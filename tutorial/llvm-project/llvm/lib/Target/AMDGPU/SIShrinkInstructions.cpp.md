# SIShrinkInstructions.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/SIShrinkInstructions.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This source file implements SIShrinkInstructions for the LLVM AMDGPU backend. It contains target-specific logic used during analysis, lowering, code generation, or pass execution. / 该源文件实现 LLVM AMDGPU 后端中的 SIShrinkInstructions 相关功能。它包含分析、降低、代码生成或 Pass 执行过程中使用的目标专用逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-39: File banner, includes, and setup
```cpp
//===-- SIShrinkInstructions.cpp - Shrink Instructions --------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
/// The pass tries to use the 32-bit encoding for instructions when possible.
//===----------------------------------------------------------------------===//
//

#include "SIShrinkInstructions.h"
#include "AMDGPU.h"
#include "GCNSubtarget.h"
#include "MCTargetDesc/AMDGPUMCTargetDesc.h"
#include "Utils/AMDGPUBaseInfo.h"
#include "llvm/ADT/Statistic.h"
#include "llvm/CodeGen/MachineFunctionPass.h"

#define DEBUG_TYPE "si-shrink-instructions"

STATISTIC(NumInstructionsShrunk,
          "Number of 64-bit instruction reduced to 32-bit.");
STATISTIC(NumLiteralConstantsFolded,
          "Number of literal constants folded into 32-bit instructions.");

using namespace llvm;

namespace {

enum ChangeKind { None, UpdateHint, UpdateInst };

class SIShrinkInstructions {
  MachineFunction *MF;
  MachineRegisterInfo *MRI;
  const GCNSubtarget *ST;
  const SIInstrInfo *TII;
  const SIRegisterInfo *TRI;
  bool IsPostRA;

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation. Main symbols: `ChangeKind`, `SIShrinkInstructions`.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。 主要符号：`ChangeKind`, `SIShrinkInstructions`。

### Lines 40-83: Declares class SIShrinkInstructionsLegacy
```cpp
  bool foldImmediates(MachineInstr &MI, bool TryToCommute = true) const;
  bool shouldShrinkTrue16(MachineInstr &MI) const;
  bool isKImmOperand(const MachineOperand &Src) const;
  bool isKUImmOperand(const MachineOperand &Src) const;
  bool isKImmOrKUImmOperand(const MachineOperand &Src, bool &IsUnsigned) const;
  void copyExtraImplicitOps(MachineInstr &NewMI, MachineInstr &MI) const;
  bool shrinkScalarCompare(MachineInstr &MI) const;
  bool shrinkMIMG(MachineInstr &MI) const;
  bool shrinkMadFma(MachineInstr &MI) const;
  ChangeKind shrinkScalarLogicOp(MachineInstr &MI) const;
  bool tryReplaceDeadSDST(MachineInstr &MI) const;
  bool instAccessReg(MachineInstr::filtered_const_mop_range &&R, Register Reg,
                     unsigned SubReg) const;
  bool instReadsReg(const MachineInstr *MI, unsigned Reg,
                    unsigned SubReg) const;
  bool instModifiesReg(const MachineInstr *MI, unsigned Reg,
                       unsigned SubReg) const;
  TargetInstrInfo::RegSubRegPair getSubRegForIndex(Register Reg, unsigned Sub,
                                                   unsigned I) const;
  void dropInstructionKeepingImpDefs(MachineInstr &MI) const;
  MachineInstr *matchSwap(MachineInstr &MovT) const;

public:
  SIShrinkInstructions() = default;
  bool run(MachineFunction &MF);
};

class SIShrinkInstructionsLegacy : public MachineFunctionPass {

public:
  static char ID;

  SIShrinkInstructionsLegacy() : MachineFunctionPass(ID) {}

  bool runOnMachineFunction(MachineFunction &MF) override;

  StringRef getPassName() const override { return "SI Shrink Instructions"; }

  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.setPreservesCFG();
    MachineFunctionPass::getAnalysisUsage(AU);
  }
};

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `SIShrinkInstructionsLegacy`, `MachineFunctionPass::getAnalysisUsage`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`SIShrinkInstructionsLegacy`, `MachineFunctionPass::getAnalysisUsage`。

### Lines 84-127: Registers LLVM passes
```cpp
} // End anonymous namespace.

INITIALIZE_PASS(SIShrinkInstructionsLegacy, DEBUG_TYPE,
                "SI Shrink Instructions", false, false)

char SIShrinkInstructionsLegacy::ID = 0;

FunctionPass *llvm::createSIShrinkInstructionsLegacyPass() {
  return new SIShrinkInstructionsLegacy();
}

/// This function checks \p MI for operands defined by a move immediate
/// instruction and then folds the literal constant into the instruction if it
/// can. This function assumes that \p MI is a VOP1, VOP2, or VOPC instructions.
bool SIShrinkInstructions::foldImmediates(MachineInstr &MI,
                                          bool TryToCommute) const {
  assert(TII->isVOP1(MI) || TII->isVOP2(MI) || TII->isVOPC(MI));

  int Src0Idx = AMDGPU::getNamedOperandIdx(MI.getOpcode(), AMDGPU::OpName::src0);

  // Try to fold Src0
  MachineOperand &Src0 = MI.getOperand(Src0Idx);
  if (Src0.isReg()) {
    Register Reg = Src0.getReg();
    if (Reg.isVirtual()) {
      MachineInstr *Def = MRI->getUniqueVRegDef(Reg);
      if (Def && Def->isMoveImmediate()) {
        MachineOperand &MovSrc = Def->getOperand(1);
        bool ConstantFolded = false;

        if (TII->isOperandLegal(MI, Src0Idx, &MovSrc)) {
          if (MovSrc.isImm()) {
            Src0.ChangeToImmediate(MovSrc.getImm());
            ConstantFolded = true;
          } else if (MovSrc.isFI()) {
            Src0.ChangeToFrameIndex(MovSrc.getIndex());
            ConstantFolded = true;
          } else if (MovSrc.isGlobal()) {
            Src0.ChangeToGA(MovSrc.getGlobal(), MovSrc.getOffset(),
                            MovSrc.getTargetFlags());
            ConstantFolded = true;
          }
        }

```
**EN:** This section connects the implementation to LLVM pass registration so the AMDGPU-specific analysis or transformation can be constructed by the pass framework. Main symbols: `llvm::createSIShrinkInstructionsLegacyPass`, `SIShrinkInstructions::foldImmediates`, `AMDGPU::getNamedOperandIdx`.
**CN:** 本节把实现接入 LLVM 的 Pass 注册机制，使 AMDGPU 专用分析或变换能够被 Pass 框架创建和调用。 主要符号：`llvm::createSIShrinkInstructionsLegacyPass`, `SIShrinkInstructions::foldImmediates`, `AMDGPU::getNamedOperandIdx`。

### Lines 128-164: Conditional logic and checks
```cpp
        if (ConstantFolded) {
          if (MRI->use_nodbg_empty(Reg))
            Def->eraseFromParent();
          ++NumLiteralConstantsFolded;
          return true;
        }
      }
    }
  }

  // We have failed to fold src0, so commute the instruction and try again.
  if (TryToCommute && MI.isCommutable()) {
    if (TII->commuteInstruction(MI)) {
      if (foldImmediates(MI, false))
        return true;

      // Commute back.
      TII->commuteInstruction(MI);
    }
  }

  return false;
}

/// Do not shrink the instruction if its registers are not expressible in the
/// shrunk encoding.
bool SIShrinkInstructions::shouldShrinkTrue16(MachineInstr &MI) const {
  for (unsigned I = 0, E = MI.getNumExplicitOperands(); I != E; ++I) {
    const MachineOperand &MO = MI.getOperand(I);
    if (MO.isReg()) {
      Register Reg = MO.getReg();
      assert(!Reg.isVirtual() && "Prior checks should ensure we only shrink "
                                 "True16 Instructions post-RA");
      if (AMDGPU::VGPR_32RegClass.contains(Reg) &&
          !AMDGPU::VGPR_32_Lo128RegClass.contains(Reg))
        return false;

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `SIShrinkInstructions::shouldShrinkTrue16`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`SIShrinkInstructions::shouldShrinkTrue16`。

### Lines 165-197: Implements SIShrinkInstructions::isKImmOperand
```cpp
      if (AMDGPU::VGPR_16RegClass.contains(Reg) &&
          !AMDGPU::VGPR_16_Lo128RegClass.contains(Reg))
        return false;
    }
  }
  return true;
}

bool SIShrinkInstructions::isKImmOperand(const MachineOperand &Src) const {
  return isInt<16>(SignExtend64(Src.getImm(), 32)) &&
         !TII->isInlineConstant(*Src.getParent(), Src.getOperandNo());
}

bool SIShrinkInstructions::isKUImmOperand(const MachineOperand &Src) const {
  return isUInt<16>(Src.getImm()) &&
         !TII->isInlineConstant(*Src.getParent(), Src.getOperandNo());
}

bool SIShrinkInstructions::isKImmOrKUImmOperand(const MachineOperand &Src,
                                                bool &IsUnsigned) const {
  if (isInt<16>(SignExtend64(Src.getImm(), 32))) {
    IsUnsigned = false;
    return !TII->isInlineConstant(Src);
  }

  if (isUInt<16>(Src.getImm())) {
    IsUnsigned = true;
    return !TII->isInlineConstant(Src);
  }

  return false;
}

```
**EN:** This section contains concrete logic for SIShrinkInstructions::isKImmOperand. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `SIShrinkInstructions::isKImmOperand`, `SIShrinkInstructions::isKUImmOperand`, `SIShrinkInstructions::isKImmOrKUImmOperand`.
**CN:** 本节包含与 SIShrinkInstructions::isKImmOperand 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`SIShrinkInstructions::isKImmOperand`, `SIShrinkInstructions::isKUImmOperand`, `SIShrinkInstructions::isKImmOrKUImmOperand`。

### Lines 198-229: Defines canModifyToInlineImmOp32
```cpp
/// \returns the opcode of an instruction a move immediate of the constant \p
/// Src can be replaced with if the constant is replaced with \p ModifiedImm.
/// i.e.
///
/// If the bitreverse of a constant is an inline immediate, reverse the
/// immediate and return the bitreverse opcode.
///
/// If the bitwise negation of a constant is an inline immediate, reverse the
/// immediate and return the bitwise not opcode.
static unsigned canModifyToInlineImmOp32(const SIInstrInfo *TII,
                                         const MachineOperand &Src,
                                         int32_t &ModifiedImm, bool Scalar) {
  if (TII->isInlineConstant(Src))
    return 0;
  int32_t SrcImm = static_cast<int32_t>(Src.getImm());

  if (!Scalar) {
    // We could handle the scalar case with here, but we would need to check
    // that SCC is not live as S_NOT_B32 clobbers it. It's probably not worth
    // it, as the reasonable values are already covered by s_movk_i32.
    ModifiedImm = ~SrcImm;
    if (TII->isInlineConstant(APInt(32, ModifiedImm, true)))
      return AMDGPU::V_NOT_B32_e32;
  }

  ModifiedImm = reverseBits<int32_t>(SrcImm);
  if (TII->isInlineConstant(APInt(32, ModifiedImm, true)))
    return Scalar ? AMDGPU::S_BREV_B32 : AMDGPU::V_BFREV_B32_e32;

  return 0;
}

```
**EN:** This section contains concrete logic for canModifyToInlineImmOp32. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 canModifyToInlineImmOp32 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 230-270: Implements SIShrinkInstructions::copyExtraImplicitOps
```cpp
/// Copy implicit register operands from specified instruction to this
/// instruction that are not part of the instruction definition.
void SIShrinkInstructions::copyExtraImplicitOps(MachineInstr &NewMI,
                                                MachineInstr &MI) const {
  MachineFunction &MF = *MI.getMF();
  for (unsigned i = MI.getDesc().getNumOperands() +
                    MI.getDesc().implicit_uses().size() +
                    MI.getDesc().implicit_defs().size(),
                e = MI.getNumOperands();
       i != e; ++i) {
    const MachineOperand &MO = MI.getOperand(i);
    if ((MO.isReg() && MO.isImplicit()) || MO.isRegMask())
      NewMI.addOperand(MF, MO);
  }
}

bool SIShrinkInstructions::shrinkScalarCompare(MachineInstr &MI) const {
  if (!ST->hasSCmpK())
    return false;

  // cmpk instructions do scc = dst <cc op> imm16, so commute the instruction to
  // get constants on the RHS.
  bool Changed = false;
  if (!MI.getOperand(0).isReg()) {
    if (TII->commuteInstruction(MI, false, 0, 1))
      Changed = true;
  }

  // cmpk requires src0 to be a register
  const MachineOperand &Src0 = MI.getOperand(0);
  if (!Src0.isReg())
    return Changed;

  MachineOperand &Src1 = MI.getOperand(1);
  if (!Src1.isImm())
    return Changed;

  int SOPKOpc = AMDGPU::getSOPKOp(MI.getOpcode());
  if (SOPKOpc == -1)
    return Changed;

```
**EN:** This section contains concrete logic for SIShrinkInstructions::copyExtraImplicitOps. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `SIShrinkInstructions::copyExtraImplicitOps`, `SIShrinkInstructions::shrinkScalarCompare`, `AMDGPU::getSOPKOp`.
**CN:** 本节包含与 SIShrinkInstructions::copyExtraImplicitOps 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`SIShrinkInstructions::copyExtraImplicitOps`, `SIShrinkInstructions::shrinkScalarCompare`, `AMDGPU::getSOPKOp`。

### Lines 271-306: Conditional logic and checks
```cpp
  // eq/ne is special because the imm16 can be treated as signed or unsigned,
  // and initially selected to the unsigned versions.
  if (SOPKOpc == AMDGPU::S_CMPK_EQ_U32 || SOPKOpc == AMDGPU::S_CMPK_LG_U32) {
    bool HasUImm;
    if (isKImmOrKUImmOperand(Src1, HasUImm)) {
      if (!HasUImm) {
        SOPKOpc = (SOPKOpc == AMDGPU::S_CMPK_EQ_U32) ?
          AMDGPU::S_CMPK_EQ_I32 : AMDGPU::S_CMPK_LG_I32;
        Src1.setImm(SignExtend32(Src1.getImm(), 32));
      }

      MI.setDesc(TII->get(SOPKOpc));
      Changed = true;
    }

    return Changed;
  }

  const MCInstrDesc &NewDesc = TII->get(SOPKOpc);

  if ((SIInstrInfo::sopkIsZext(SOPKOpc) && isKUImmOperand(Src1)) ||
      (!SIInstrInfo::sopkIsZext(SOPKOpc) && isKImmOperand(Src1))) {
    if (!SIInstrInfo::sopkIsZext(SOPKOpc))
      Src1.setImm(SignExtend64(Src1.getImm(), 32));
    MI.setDesc(NewDesc);
    Changed = true;
  }
  return Changed;
}

// Shrink NSA encoded instructions with contiguous VGPRs to non-NSA encoding.
bool SIShrinkInstructions::shrinkMIMG(MachineInstr &MI) const {
  const AMDGPU::MIMGInfo *Info = AMDGPU::getMIMGInfo(MI.getOpcode());
  if (!Info)
    return false;

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `SIInstrInfo::sopkIsZext`, `SIShrinkInstructions::shrinkMIMG`, `AMDGPU::getMIMGInfo`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`SIInstrInfo::sopkIsZext`, `SIShrinkInstructions::shrinkMIMG`, `AMDGPU::getMIMGInfo`。

### Lines 307-350: Switch-based control flow
```cpp
  uint8_t NewEncoding;
  switch (Info->MIMGEncoding) {
  case AMDGPU::MIMGEncGfx10NSA:
    NewEncoding = AMDGPU::MIMGEncGfx10Default;
    break;
  case AMDGPU::MIMGEncGfx11NSA:
    NewEncoding = AMDGPU::MIMGEncGfx11Default;
    break;
  default:
    return false;
  }

  int VAddr0Idx =
      AMDGPU::getNamedOperandIdx(MI.getOpcode(), AMDGPU::OpName::vaddr0);
  unsigned NewAddrDwords = Info->VAddrDwords;
  const TargetRegisterClass *RC;

  if (Info->VAddrDwords == 2) {
    RC = &AMDGPU::VReg_64RegClass;
  } else if (Info->VAddrDwords == 3) {
    RC = &AMDGPU::VReg_96RegClass;
  } else if (Info->VAddrDwords == 4) {
    RC = &AMDGPU::VReg_128RegClass;
  } else if (Info->VAddrDwords == 5) {
    RC = &AMDGPU::VReg_160RegClass;
  } else if (Info->VAddrDwords == 6) {
    RC = &AMDGPU::VReg_192RegClass;
  } else if (Info->VAddrDwords == 7) {
    RC = &AMDGPU::VReg_224RegClass;
  } else if (Info->VAddrDwords == 8) {
    RC = &AMDGPU::VReg_256RegClass;
  } else if (Info->VAddrDwords == 9) {
    RC = &AMDGPU::VReg_288RegClass;
  } else if (Info->VAddrDwords == 10) {
    RC = &AMDGPU::VReg_320RegClass;
  } else if (Info->VAddrDwords == 11) {
    RC = &AMDGPU::VReg_352RegClass;
  } else if (Info->VAddrDwords == 12) {
    RC = &AMDGPU::VReg_384RegClass;
  } else {
    RC = &AMDGPU::VReg_512RegClass;
    NewAddrDwords = 16;
  }

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `AMDGPU::getNamedOperandIdx`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`AMDGPU::getNamedOperandIdx`。

### Lines 351-381: Declares getNSAMaxSize
```cpp
  unsigned VgprBase = 0;
  unsigned NextVgpr = 0;
  bool IsUndef = true;
  bool IsKill = NewAddrDwords == Info->VAddrDwords;
  const unsigned NSAMaxSize = ST->getNSAMaxSize();
  const bool IsPartialNSA = NewAddrDwords > NSAMaxSize;
  const unsigned EndVAddr = IsPartialNSA ? NSAMaxSize : Info->VAddrOperands;
  for (unsigned Idx = 0; Idx < EndVAddr; ++Idx) {
    const MachineOperand &Op = MI.getOperand(VAddr0Idx + Idx);
    unsigned Vgpr = TRI->getHWRegIndex(Op.getReg());
    unsigned Dwords = TRI->getRegSizeInBits(Op.getReg(), *MRI) / 32;
    assert(Dwords > 0 && "Un-implemented for less than 32 bit regs");

    if (Idx == 0) {
      VgprBase = Vgpr;
      NextVgpr = Vgpr + Dwords;
    } else if (Vgpr == NextVgpr) {
      NextVgpr = Vgpr + Dwords;
    } else {
      return false;
    }

    if (!Op.isUndef())
      IsUndef = false;
    if (!Op.isKill())
      IsKill = false;
  }

  if (VgprBase + NewAddrDwords > 256)
    return false;

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。

### Lines 382-421: Implements AMDGPU::getNamedOperandIdx
```cpp
  // Further check for implicit tied operands - this may be present if TFE is
  // enabled
  int TFEIdx = AMDGPU::getNamedOperandIdx(MI.getOpcode(), AMDGPU::OpName::tfe);
  int LWEIdx = AMDGPU::getNamedOperandIdx(MI.getOpcode(), AMDGPU::OpName::lwe);
  unsigned TFEVal = (TFEIdx == -1) ? 0 : MI.getOperand(TFEIdx).getImm();
  unsigned LWEVal = (LWEIdx == -1) ? 0 : MI.getOperand(LWEIdx).getImm();
  int ToUntie = -1;
  if (TFEVal || LWEVal) {
    // TFE/LWE is enabled so we need to deal with an implicit tied operand
    for (unsigned i = LWEIdx + 1, e = MI.getNumOperands(); i != e; ++i) {
      if (MI.getOperand(i).isReg() && MI.getOperand(i).isTied() &&
          MI.getOperand(i).isImplicit()) {
        // This is the tied operand
        assert(
            ToUntie == -1 &&
            "found more than one tied implicit operand when expecting only 1");
        ToUntie = i;
        MI.untieRegOperand(ToUntie);
      }
    }
  }

  unsigned NewOpcode = AMDGPU::getMIMGOpcode(Info->BaseOpcode, NewEncoding,
                                             Info->VDataDwords, NewAddrDwords);
  MI.setDesc(TII->get(NewOpcode));
  MI.getOperand(VAddr0Idx).setReg(RC->getRegister(VgprBase));
  MI.getOperand(VAddr0Idx).setIsUndef(IsUndef);
  MI.getOperand(VAddr0Idx).setIsKill(IsKill);

  for (unsigned i = 1; i < EndVAddr; ++i)
    MI.removeOperand(VAddr0Idx + 1);

  if (ToUntie >= 0) {
    MI.tieOperands(
        AMDGPU::getNamedOperandIdx(MI.getOpcode(), AMDGPU::OpName::vdata),
        ToUntie - (EndVAddr - 1));
  }
  return true;
}

```
**EN:** This section contains concrete logic for AMDGPU::getNamedOperandIdx. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPU::getNamedOperandIdx`, `AMDGPU::getMIMGOpcode`.
**CN:** 本节包含与 AMDGPU::getNamedOperandIdx 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPU::getNamedOperandIdx`, `AMDGPU::getMIMGOpcode`。

### Lines 422-452: Implements SIShrinkInstructions::shrinkMadFma
```cpp
// Shrink MAD to MADAK/MADMK and FMA to FMAAK/FMAMK.
bool SIShrinkInstructions::shrinkMadFma(MachineInstr &MI) const {
  // Pre-GFX10 VOP3 instructions like MAD/FMA cannot take a literal operand so
  // there is no reason to try to shrink them.
  if (!ST->hasVOP3Literal())
    return false;

  // There is no advantage to doing this pre-RA.
  if (!IsPostRA)
    return false;

  if (TII->hasAnyModifiersSet(MI))
    return false;

  const unsigned Opcode = MI.getOpcode();
  MachineOperand &Src0 = *TII->getNamedOperand(MI, AMDGPU::OpName::src0);
  MachineOperand &Src1 = *TII->getNamedOperand(MI, AMDGPU::OpName::src1);
  MachineOperand &Src2 = *TII->getNamedOperand(MI, AMDGPU::OpName::src2);
  unsigned NewOpcode = AMDGPU::INSTRUCTION_LIST_END;

  bool Swap;

  // Detect "Dst = VSrc * VGPR + Imm" and convert to AK form.
  if (Src2.isImm() && !TII->isInlineConstant(Src2)) {
    if (Src1.isReg() && TRI->isVGPR(*MRI, Src1.getReg()))
      Swap = false;
    else if (Src0.isReg() && TRI->isVGPR(*MRI, Src0.getReg()))
      Swap = true;
    else
      return false;

```
**EN:** This section contains concrete logic for SIShrinkInstructions::shrinkMadFma. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `SIShrinkInstructions::shrinkMadFma`.
**CN:** 本节包含与 SIShrinkInstructions::shrinkMadFma 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`SIShrinkInstructions::shrinkMadFma`。

### Lines 453-490: Switch-based control flow
```cpp
    switch (Opcode) {
    default:
      llvm_unreachable("Unexpected mad/fma opcode!");
    case AMDGPU::V_MAD_F32_e64:
      NewOpcode = AMDGPU::V_MADAK_F32;
      break;
    case AMDGPU::V_FMA_F32_e64:
      NewOpcode = AMDGPU::V_FMAAK_F32;
      break;
    case AMDGPU::V_MAD_F16_e64:
      NewOpcode = AMDGPU::V_MADAK_F16;
      break;
    case AMDGPU::V_FMA_F16_e64:
    case AMDGPU::V_FMA_F16_gfx9_e64:
      NewOpcode = AMDGPU::V_FMAAK_F16;
      break;
    case AMDGPU::V_FMA_F16_gfx9_t16_e64:
      NewOpcode = AMDGPU::V_FMAAK_F16_t16;
      break;
    case AMDGPU::V_FMA_F16_gfx9_fake16_e64:
      NewOpcode = AMDGPU::V_FMAAK_F16_fake16;
      break;
    case AMDGPU::V_FMA_F64_e64:
      if (ST->hasFmaakFmamkF64Insts())
        NewOpcode = AMDGPU::V_FMAAK_F64;
      break;
    }
  }

  // Detect "Dst = VSrc * Imm + VGPR" and convert to MK form.
  if (Src2.isReg() && TRI->isVGPR(*MRI, Src2.getReg())) {
    if (Src1.isImm() && !TII->isInlineConstant(Src1))
      Swap = false;
    else if (Src0.isImm() && !TII->isInlineConstant(Src0))
      Swap = true;
    else
      return false;

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 491-525: Switch-based control flow
```cpp
    switch (Opcode) {
    default:
      llvm_unreachable("Unexpected mad/fma opcode!");
    case AMDGPU::V_MAD_F32_e64:
      NewOpcode = AMDGPU::V_MADMK_F32;
      break;
    case AMDGPU::V_FMA_F32_e64:
      NewOpcode = AMDGPU::V_FMAMK_F32;
      break;
    case AMDGPU::V_MAD_F16_e64:
      NewOpcode = AMDGPU::V_MADMK_F16;
      break;
    case AMDGPU::V_FMA_F16_e64:
    case AMDGPU::V_FMA_F16_gfx9_e64:
      NewOpcode = AMDGPU::V_FMAMK_F16;
      break;
    case AMDGPU::V_FMA_F16_gfx9_t16_e64:
      NewOpcode = AMDGPU::V_FMAMK_F16_t16;
      break;
    case AMDGPU::V_FMA_F16_gfx9_fake16_e64:
      NewOpcode = AMDGPU::V_FMAMK_F16_fake16;
      break;
    case AMDGPU::V_FMA_F64_e64:
      if (ST->hasFmaakFmamkF64Insts())
        NewOpcode = AMDGPU::V_FMAMK_F64;
      break;
    }
  }

  if (NewOpcode == AMDGPU::INSTRUCTION_LIST_END)
    return false;

  if (AMDGPU::isTrue16Inst(NewOpcode) && !shouldShrinkTrue16(MI))
    return false;

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `AMDGPU::isTrue16Inst`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`AMDGPU::isTrue16Inst`。

### Lines 526-563: Type declarations and aliases
```cpp
  if (Swap) {
    // Swap Src0 and Src1 by building a new instruction.
    BuildMI(*MI.getParent(), MI, MI.getDebugLoc(), TII->get(NewOpcode),
            MI.getOperand(0).getReg())
        .add(Src1)
        .add(Src0)
        .add(Src2)
        .setMIFlags(MI.getFlags());
    MI.eraseFromParent();
  } else {
    TII->removeModOperands(MI);
    MI.setDesc(TII->get(NewOpcode));
  }
  return true;
}

/// Attempt to shrink AND/OR/XOR operations requiring non-inlineable literals.
/// For AND or OR, try using S_BITSET{0,1} to clear or set bits.
/// If the inverse of the immediate is legal, use ANDN2, ORN2 or
/// XNOR (as a ^ b == ~(a ^ ~b)).
/// \return ChangeKind::None if no changes were made.
///         ChangeKind::UpdateHint if regalloc hints were updated.
///         ChangeKind::UpdateInst if the instruction was modified.
ChangeKind SIShrinkInstructions::shrinkScalarLogicOp(MachineInstr &MI) const {
  unsigned Opc = MI.getOpcode();
  const MachineOperand *Dest = &MI.getOperand(0);
  MachineOperand *Src0 = &MI.getOperand(1);
  MachineOperand *Src1 = &MI.getOperand(2);
  MachineOperand *SrcReg = Src0;
  MachineOperand *SrcImm = Src1;

  if (!SrcImm->isImm() ||
      AMDGPU::isInlinableLiteral32(SrcImm->getImm(), ST->hasInv2PiInlineImm()))
    return ChangeKind::None;

  uint32_t Imm = static_cast<uint32_t>(SrcImm->getImm());
  uint32_t NewImm = 0;

```
**EN:** This section introduces supporting types or aliases that simplify later declarations and backend logic. Main symbols: `SIShrinkInstructions::shrinkScalarLogicOp`, `AMDGPU::isInlinableLiteral32`.
**CN:** 本节引入辅助类型或别名，以简化后续声明与后端逻辑。 主要符号：`SIShrinkInstructions::shrinkScalarLogicOp`, `AMDGPU::isInlinableLiteral32`。

### Lines 564-597: Conditional logic and checks
```cpp
  if (Opc == AMDGPU::S_AND_B32) {
    if (isPowerOf2_32(~Imm) &&
        MI.findRegisterDefOperand(AMDGPU::SCC, /*TRI=*/nullptr)->isDead()) {
      NewImm = llvm::countr_one(Imm);
      Opc = AMDGPU::S_BITSET0_B32;
    } else if (AMDGPU::isInlinableLiteral32(~Imm, ST->hasInv2PiInlineImm())) {
      NewImm = ~Imm;
      Opc = AMDGPU::S_ANDN2_B32;
    }
  } else if (Opc == AMDGPU::S_OR_B32) {
    if (isPowerOf2_32(Imm) &&
        MI.findRegisterDefOperand(AMDGPU::SCC, /*TRI=*/nullptr)->isDead()) {
      NewImm = llvm::countr_zero(Imm);
      Opc = AMDGPU::S_BITSET1_B32;
    } else if (AMDGPU::isInlinableLiteral32(~Imm, ST->hasInv2PiInlineImm())) {
      NewImm = ~Imm;
      Opc = AMDGPU::S_ORN2_B32;
    }
  } else if (Opc == AMDGPU::S_XOR_B32) {
    if (AMDGPU::isInlinableLiteral32(~Imm, ST->hasInv2PiInlineImm())) {
      NewImm = ~Imm;
      Opc = AMDGPU::S_XNOR_B32;
    }
  } else {
    llvm_unreachable("unexpected opcode");
  }

  if (NewImm != 0) {
    if (Dest->getReg().isVirtual() && SrcReg->isReg()) {
      MRI->setRegAllocationHint(Dest->getReg(), 0, SrcReg->getReg());
      MRI->setRegAllocationHint(SrcReg->getReg(), 0, Dest->getReg());
      return ChangeKind::UpdateHint;
    }

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `llvm::countr_one`, `AMDGPU::isInlinableLiteral32`, `llvm::countr_zero`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`llvm::countr_one`, `AMDGPU::isInlinableLiteral32`, `llvm::countr_zero`。

### Lines 598-638: Conditional logic and checks
```cpp
    if (SrcReg->isReg() && SrcReg->getReg() == Dest->getReg()) {
      const bool IsUndef = SrcReg->isUndef();
      const bool IsKill = SrcReg->isKill();
      TII->mutateAndCleanupImplicit(MI, TII->get(Opc));
      if (Opc == AMDGPU::S_BITSET0_B32 ||
          Opc == AMDGPU::S_BITSET1_B32) {
        Src0->ChangeToImmediate(NewImm);
        // Remove the immediate and add the tied input.
        MI.getOperand(2).ChangeToRegister(Dest->getReg(), /*IsDef*/ false,
                                          /*isImp*/ false, IsKill,
                                          /*isDead*/ false, IsUndef);
        MI.tieOperands(0, 2);
      } else {
        SrcImm->setImm(NewImm);
      }
      return ChangeKind::UpdateInst;
    }
  }

  return ChangeKind::None;
}

// This is the same as MachineInstr::readsRegister/modifiesRegister except
// it takes subregs into account.
bool SIShrinkInstructions::instAccessReg(
    MachineInstr::filtered_const_mop_range &&R, Register Reg,
    unsigned SubReg) const {
  for (const MachineOperand &MO : R) {
    if (Reg.isPhysical() && MO.getReg().isPhysical()) {
      if (TRI->regsOverlap(Reg, MO.getReg()))
        return true;
    } else if (MO.getReg() == Reg && Reg.isVirtual()) {
      LaneBitmask Overlap = TRI->getSubRegIndexLaneMask(SubReg) &
                            TRI->getSubRegIndexLaneMask(MO.getSubReg());
      if (Overlap.any())
        return true;
    }
  }
  return false;
}

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `SIShrinkInstructions::instAccessReg`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`SIShrinkInstructions::instAccessReg`。

### Lines 639-678: Implements SIShrinkInstructions::instReadsReg
```cpp
bool SIShrinkInstructions::instReadsReg(const MachineInstr *MI, unsigned Reg,
                                        unsigned SubReg) const {
  return instAccessReg(MI->all_uses(), Reg, SubReg);
}

bool SIShrinkInstructions::instModifiesReg(const MachineInstr *MI, unsigned Reg,
                                           unsigned SubReg) const {
  return instAccessReg(MI->all_defs(), Reg, SubReg);
}

TargetInstrInfo::RegSubRegPair
SIShrinkInstructions::getSubRegForIndex(Register Reg, unsigned Sub,
                                        unsigned I) const {
  if (TRI->getRegSizeInBits(Reg, *MRI) != 32) {
    if (Reg.isPhysical()) {
      Reg = TRI->getSubReg(Reg, TRI->getSubRegFromChannel(I));
    } else {
      Sub = TRI->getSubRegFromChannel(I + TRI->getChannelFromSubReg(Sub));
    }
  }
  return TargetInstrInfo::RegSubRegPair(Reg, Sub);
}

void SIShrinkInstructions::dropInstructionKeepingImpDefs(
    MachineInstr &MI) const {
  for (unsigned i = MI.getDesc().getNumOperands() +
                    MI.getDesc().implicit_uses().size() +
                    MI.getDesc().implicit_defs().size(),
                e = MI.getNumOperands();
       i != e; ++i) {
    const MachineOperand &Op = MI.getOperand(i);
    if (!Op.isDef())
      continue;
    BuildMI(*MI.getParent(), MI.getIterator(), MI.getDebugLoc(),
            TII->get(AMDGPU::IMPLICIT_DEF), Op.getReg());
  }

  MI.eraseFromParent();
}

```
**EN:** This section contains concrete logic for SIShrinkInstructions::instReadsReg. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `SIShrinkInstructions::instReadsReg`, `SIShrinkInstructions::instModifiesReg`, `SIShrinkInstructions::getSubRegForIndex`.
**CN:** 本节包含与 SIShrinkInstructions::instReadsReg 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`SIShrinkInstructions::instReadsReg`, `SIShrinkInstructions::instModifiesReg`, `SIShrinkInstructions::getSubRegForIndex`。

### Lines 679-720: Conditional logic and checks
```cpp
// Match:
// mov t, x
// mov x, y
// mov y, t
//
// =>
//
// mov t, x (t is potentially dead and move eliminated)
// v_swap_b32 x, y
//
// Returns next valid instruction pointer if was able to create v_swap_b32.
//
// This shall not be done too early not to prevent possible folding which may
// remove matched moves, and this should preferably be done before RA to
// release saved registers and also possibly after RA which can insert copies
// too.
//
// This is really just a generic peephole that is not a canonical shrinking,
// although requirements match the pass placement and it reduces code size too.
MachineInstr *SIShrinkInstructions::matchSwap(MachineInstr &MovT) const {
  assert(MovT.getOpcode() == AMDGPU::V_MOV_B32_e32 ||
         MovT.getOpcode() == AMDGPU::V_MOV_B16_t16_e32 ||
         MovT.getOpcode() == AMDGPU::COPY);

  Register T = MovT.getOperand(0).getReg();
  unsigned Tsub = MovT.getOperand(0).getSubReg();
  MachineOperand &Xop = MovT.getOperand(1);

  if (!Xop.isReg())
    return nullptr;
  Register X = Xop.getReg();
  unsigned Xsub = Xop.getSubReg();
  Register Y;
  unsigned Ysub;

  unsigned Size = TII->getOpSize(MovT, 0);

  // We can't match v_swap_b16 pre-RA, because VGPR_16_Lo128 registers
  // are not allocatble.
  if (Size == 2 && X.isVirtual())
    return nullptr;

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `SIShrinkInstructions::matchSwap`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`SIShrinkInstructions::matchSwap`。

### Lines 721-764: Conditional logic and checks
```cpp
  if (!TRI->isVGPR(*MRI, X))
    return nullptr;

  const unsigned SearchLimit = 16;
  unsigned Count = 0;

  MachineInstr *MovX = nullptr;
  MachineInstr *InsertionPt = nullptr;
  MachineInstr *MovY = nullptr;

  for (auto Iter = std::next(MovT.getIterator()),
            E = MovT.getParent()->instr_end();
       Iter != E && Count < SearchLimit; ++Iter) {
    if (Iter->isDebugInstr())
      continue;
    ++Count;

    if (!MovX) {
      // Search for mov x, y.
      if ((Iter->getOpcode() == AMDGPU::V_MOV_B32_e32 ||
           Iter->getOpcode() == AMDGPU::V_MOV_B16_t16_e32 ||
           Iter->getOpcode() == AMDGPU::COPY) &&
          Iter->getOperand(0).getReg() == X &&
          Iter->getOperand(0).getSubReg() == Xsub &&
          Iter->getOperand(1).isReg()) {
        MovX = &*Iter;
        Y = MovX->getOperand(1).getReg();
        Ysub = MovX->getOperand(1).getSubReg();
      } else if (instModifiesReg(&*Iter, X, Xsub)) {
        // Writes to x are not allowed until mov x, y has been found
        return nullptr;
      }
    } else {
      // mov x, y has been found.
      // Search for mov y, t.
      if ((Iter->getOpcode() == AMDGPU::V_MOV_B32_e32 ||
           Iter->getOpcode() == AMDGPU::V_MOV_B16_t16_e32 ||
           Iter->getOpcode() == AMDGPU::COPY) &&
          Iter->getOperand(0).getReg() == Y &&
          Iter->getOperand(0).getSubReg() == Ysub &&
          Iter->getOperand(1).isReg() && Iter->getOperand(1).getReg() == T &&
          Iter->getOperand(1).getSubReg() == Tsub) {
        MovY = &*Iter;
        break;
```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `std::next`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`std::next`。

### Lines 765-797: Conditional logic and checks
```cpp
      }

      // Effectively, mov x, y must be moved downward
      // and mov y, t must be moved upward so that they can be fused into a
      // swap. A write to y creates a barrier that prevents the two moves from
      // being moved adjacent to each other.
      if (instModifiesReg(&*Iter, Y, Ysub))
        return nullptr;

      // Reads or writes to x prevent mov x, y from being moved farther
      // downward. Select this to be the insertion point.
      if (!InsertionPt &&
          (instReadsReg(&*Iter, X, Xsub) || instModifiesReg(&*Iter, X, Xsub))) {
        InsertionPt = &*Iter;
      }
      // If the insertion point has been found, then mov y, t must be moved
      // upward past all subsequent instructions.  A read of y will block this
      // movement.
      if (InsertionPt) {
        if (instReadsReg(&*Iter, Y, Ysub))
          return nullptr;
      }
    }

    if (instModifiesReg(&*Iter, T, Tsub))
      return nullptr;
  }
  if (MovY) {
    LLVM_DEBUG(dbgs() << "Matched v_swap:\n" << MovT << *MovX << *MovY);

    MachineBasicBlock &MBB = *MovT.getParent();
    SmallVector<MachineInstr *, 4> Swaps;

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 798-835: Conditional logic and checks
```cpp
    if (!InsertionPt)
      InsertionPt = MovY;
    if (Size == 2) {
      auto *MIB = BuildMI(MBB, InsertionPt->getIterator(), MovT.getDebugLoc(),
                          TII->get(AMDGPU::V_SWAP_B16))
                      .addDef(X)
                      .addDef(Y)
                      .addReg(Y)
                      .addReg(X)
                      .getInstr();
      Swaps.push_back(MIB);
    } else {
      assert(Size > 0 && Size % 4 == 0);
      for (unsigned I = 0; I < Size / 4; ++I) {
        TargetInstrInfo::RegSubRegPair X1, Y1;
        X1 = getSubRegForIndex(X, Xsub, I);
        Y1 = getSubRegForIndex(Y, Ysub, I);
        auto *MIB = BuildMI(MBB, InsertionPt->getIterator(), MovT.getDebugLoc(),
                            TII->get(AMDGPU::V_SWAP_B32))
                        .addDef(X1.Reg, {}, X1.SubReg)
                        .addDef(Y1.Reg, {}, Y1.SubReg)
                        .addReg(Y1.Reg, {}, Y1.SubReg)
                        .addReg(X1.Reg, {}, X1.SubReg)
                        .getInstr();
        Swaps.push_back(MIB);
      }
    }
    // Drop implicit EXEC.
    if (MovX->hasRegisterImplicitUseOperand(AMDGPU::EXEC)) {
      for (MachineInstr *Swap : Swaps) {
        Swap->removeOperand(Swap->getNumExplicitOperands());
        Swap->copyImplicitOps(*MBB.getParent(), *MovX);
      }
    }
    MovX->eraseFromParent();
    dropInstructionKeepingImpDefs(*MovY);
    MachineInstr *Next = &*std::next(MovT.getIterator());

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `std::next`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`std::next`。

### Lines 836-877: Conditional logic and checks
```cpp
    if (T.isVirtual() && MRI->use_nodbg_empty(T)) {
      dropInstructionKeepingImpDefs(MovT);
    } else {
      Xop.setIsKill(false);
      for (int I = MovT.getNumImplicitOperands() - 1; I >= 0; --I ) {
        unsigned OpNo = MovT.getNumExplicitOperands() + I;
        const MachineOperand &Op = MovT.getOperand(OpNo);
        if (Op.isKill() && TRI->regsOverlap(X, Op.getReg()))
          MovT.removeOperand(OpNo);
      }
    }

    return Next;
  }
  return nullptr;
}

// If an instruction has dead sdst replace it with NULL register on gfx1030+
bool SIShrinkInstructions::tryReplaceDeadSDST(MachineInstr &MI) const {
  if (!ST->hasGFX10_3Insts())
    return false;

  MachineOperand *Op = TII->getNamedOperand(MI, AMDGPU::OpName::sdst);
  if (!Op)
    return false;
  Register SDstReg = Op->getReg();
  if (SDstReg.isPhysical() || !MRI->use_nodbg_empty(SDstReg))
    return false;

  Op->setReg(ST->isWave32() ? AMDGPU::SGPR_NULL : AMDGPU::SGPR_NULL64);
  return true;
}

bool SIShrinkInstructions::run(MachineFunction &MF) {

  this->MF = &MF;
  MRI = &MF.getRegInfo();
  ST = &MF.getSubtarget<GCNSubtarget>();
  TII = ST->getInstrInfo();
  TRI = &TII->getRegisterInfo();
  IsPostRA = MF.getProperties().hasNoVRegs();

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `SIShrinkInstructions::tryReplaceDeadSDST`, `SIShrinkInstructions::run`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`SIShrinkInstructions::tryReplaceDeadSDST`, `SIShrinkInstructions::run`。

### Lines 878-918: Declares isWave32
```cpp
  unsigned VCCReg = ST->isWave32() ? AMDGPU::VCC_LO : AMDGPU::VCC;
  bool Changed = false;

  for (MachineBasicBlock &MBB : MF) {
    MachineBasicBlock::iterator I, Next;
    for (I = MBB.begin(); I != MBB.end(); I = Next) {
      Next = std::next(I);
      MachineInstr &MI = *I;

      if (MI.getOpcode() == AMDGPU::V_MOV_B32_e32) {
        // If this has a literal constant source that is the same as the
        // reversed bits of an inline immediate, replace with a bitreverse of
        // that constant. This saves 4 bytes in the common case of materializing
        // sign bits.

        // Test if we are after regalloc. We only want to do this after any
        // optimizations happen because this will confuse them.
        MachineOperand &Src = MI.getOperand(1);
        if (Src.isImm() && IsPostRA) {
          int32_t ModImm;
          unsigned ModOpcode =
              canModifyToInlineImmOp32(TII, Src, ModImm, /*Scalar=*/false);
          if (ModOpcode != 0) {
            MI.setDesc(TII->get(ModOpcode));
            Src.setImm(static_cast<int64_t>(ModImm));
            Changed = true;
            continue;
          }
        }
      }

      if (ST->hasSwap() && (MI.getOpcode() == AMDGPU::V_MOV_B32_e32 ||
                            MI.getOpcode() == AMDGPU::V_MOV_B16_t16_e32 ||
                            MI.getOpcode() == AMDGPU::COPY)) {
        if (auto *NextMI = matchSwap(MI)) {
          Next = NextMI->getIterator();
          Changed = true;
          continue;
        }
      }

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend. Main symbols: `std::next`.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。 主要符号：`std::next`。

### Lines 919-944: Conditional logic and checks
```cpp
      // Shrink scalar logic operations.
      if (MI.getOpcode() == AMDGPU::S_AND_B32 ||
          MI.getOpcode() == AMDGPU::S_OR_B32 ||
          MI.getOpcode() == AMDGPU::S_XOR_B32) {
        ChangeKind CK = shrinkScalarLogicOp(MI);
        if (CK == ChangeKind::UpdateHint)
          continue;
        Changed |= (CK == ChangeKind::UpdateInst);
      }

      // Try to use S_ADDK_I32 and S_MULK_I32.
      if (MI.getOpcode() == AMDGPU::S_ADD_I32 ||
          MI.getOpcode() == AMDGPU::S_MUL_I32 ||
          (MI.getOpcode() == AMDGPU::S_OR_B32 &&
           MI.getFlag(MachineInstr::MIFlag::Disjoint))) {
        const MachineOperand *Dest = &MI.getOperand(0);
        MachineOperand *Src0 = &MI.getOperand(1);
        MachineOperand *Src1 = &MI.getOperand(2);

        if (!Src0->isReg() && Src1->isReg()) {
          if (TII->commuteInstruction(MI, false, 1, 2)) {
            std::swap(Src0, Src1);
            Changed = true;
          }
        }

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `std::swap`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`std::swap`。

### Lines 945-976: Conditional logic and checks
```cpp
        // FIXME: This could work better if hints worked with subregisters. If
        // we have a vector add of a constant, we usually don't get the correct
        // allocation due to the subregister usage.
        if (Dest->getReg().isVirtual() && Src0->isReg()) {
          MRI->setRegAllocationHint(Dest->getReg(), 0, Src0->getReg());
          MRI->setRegAllocationHint(Src0->getReg(), 0, Dest->getReg());
          continue;
        }
        if (Src0->isReg() && Src0->getReg() == Dest->getReg()) {
          if (Src1->isImm() && isKImmOperand(*Src1)) {
            unsigned Opc = (MI.getOpcode() == AMDGPU::S_MUL_I32)
                               ? AMDGPU::S_MULK_I32
                               : AMDGPU::S_ADDK_I32;
            Src1->setImm(SignExtend64(Src1->getImm(), 32));
            MI.setDesc(TII->get(Opc));
            MI.tieOperands(0, 1);
            Changed = true;
          }
        }
      }

      // Try to use s_cmpk_*
      if (MI.isCompare() && TII->isSOPC(MI)) {
        Changed |= shrinkScalarCompare(MI);
        continue;
      }

      // Try to use S_MOVK_I32, which will save 4 bytes for small immediates.
      if (MI.getOpcode() == AMDGPU::S_MOV_B32) {
        const MachineOperand &Dst = MI.getOperand(0);
        MachineOperand &Src = MI.getOperand(1);

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 977-1016: Conditional logic and checks
```cpp
        if (Src.isImm() && Dst.getReg().isPhysical()) {
          unsigned ModOpc;
          int32_t ModImm;
          if (isKImmOperand(Src)) {
            MI.setDesc(TII->get(AMDGPU::S_MOVK_I32));
            Src.setImm(SignExtend64(Src.getImm(), 32));
            Changed = true;
          } else if ((ModOpc = canModifyToInlineImmOp32(TII, Src, ModImm,
                                                        /*Scalar=*/true))) {
            MI.setDesc(TII->get(ModOpc));
            Src.setImm(static_cast<int64_t>(ModImm));
            Changed = true;
          }
        }

        continue;
      }

      if (IsPostRA && TII->isMIMG(MI.getOpcode()) &&
          ST->getGeneration() >= AMDGPUSubtarget::GFX10) {
        Changed |= shrinkMIMG(MI);
        continue;
      }

      if (!TII->isVOP3(MI))
        continue;

      if (MI.getOpcode() == AMDGPU::V_MAD_F32_e64 ||
          MI.getOpcode() == AMDGPU::V_FMA_F32_e64 ||
          MI.getOpcode() == AMDGPU::V_MAD_F16_e64 ||
          MI.getOpcode() == AMDGPU::V_FMA_F16_e64 ||
          MI.getOpcode() == AMDGPU::V_FMA_F16_gfx9_e64 ||
          MI.getOpcode() == AMDGPU::V_FMA_F16_gfx9_t16_e64 ||
          MI.getOpcode() == AMDGPU::V_FMA_F16_gfx9_fake16_e64 ||
          (MI.getOpcode() == AMDGPU::V_FMA_F64_e64 &&
           ST->hasFmaakFmamkF64Insts())) {
        Changed |= shrinkMadFma(MI);
        continue;
      }

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 1017-1040: Conditional logic and checks
```cpp
      // If there is no chance we will shrink it and use VCC as sdst to get
      // a 32 bit form try to replace dead sdst with NULL.
      if (TII->isVOP3(MI.getOpcode())) {
        Changed |= tryReplaceDeadSDST(MI);
        if (!TII->hasVALU32BitEncoding(MI.getOpcode())) {
          continue;
        }
      }

      if (!TII->canShrink(MI, *MRI)) {
        // Try commuting the instruction and see if that enables us to shrink
        // it.
        if (!MI.isCommutable() || !TII->commuteInstruction(MI) ||
            !TII->canShrink(MI, *MRI)) {
          Changed |= tryReplaceDeadSDST(MI);
          continue;
        }

        // Operands were commuted.
        Changed = true;
      }

      int Op32 = AMDGPU::getVOPe32(MI.getOpcode());

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `AMDGPU::getVOPe32`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`AMDGPU::getVOPe32`。

### Lines 1041-1084: Type declarations and aliases
```cpp
      if (TII->isVOPC(Op32)) {
        MachineOperand &Op0 = MI.getOperand(0);
        if (Op0.isReg()) {
          // Exclude VOPCX instructions as these don't explicitly write a
          // dst.
          Register DstReg = Op0.getReg();
          if (DstReg.isVirtual()) {
            // VOPC instructions can only write to the VCC register. We can't
            // force them to use VCC here, because this is only one register and
            // cannot deal with sequences which would require multiple copies of
            // VCC, e.g. S_AND_B64 (vcc = V_CMP_...), (vcc = V_CMP_...)
            //
            // So, instead of forcing the instruction to write to VCC, we
            // provide a hint to the register allocator to use VCC and then we
            // will run this pass again after RA and shrink it if it outputs to
            // VCC.
            MRI->setRegAllocationHint(DstReg, 0, VCCReg);
            continue;
          }
          if (DstReg != VCCReg)
            continue;
        }
      }

      if (Op32 == AMDGPU::V_CNDMASK_B32_e32) {
        // We shrink V_CNDMASK_B32_e64 using regalloc hints like we do for VOPC
        // instructions.
        const MachineOperand *Src2 =
            TII->getNamedOperand(MI, AMDGPU::OpName::src2);
        if (!Src2->isReg())
          continue;
        Register SReg = Src2->getReg();
        if (SReg.isVirtual()) {
          MRI->setRegAllocationHint(SReg, 0, VCCReg);
          continue;
        }
        if (SReg != VCCReg)
          continue;
      }

      // Check for the bool flag output for instructions like V_ADD_I32_e64.
      const MachineOperand *SDst = TII->getNamedOperand(MI,
                                                        AMDGPU::OpName::sdst);

```
**EN:** This section introduces supporting types or aliases that simplify later declarations and backend logic.
**CN:** 本节引入辅助类型或别名，以简化后续声明与后端逻辑。

### Lines 1085-1128: Conditional logic and checks
```cpp
      if (SDst) {
        bool Next = false;

        if (SDst->getReg() != VCCReg) {
          if (SDst->getReg().isVirtual())
            MRI->setRegAllocationHint(SDst->getReg(), 0, VCCReg);
          Next = true;
        }

        // All of the instructions with carry outs also have an SGPR input in
        // src2.
        const MachineOperand *Src2 = TII->getNamedOperand(MI,
                                                          AMDGPU::OpName::src2);
        if (Src2 && Src2->getReg() != VCCReg) {
          if (Src2->getReg().isVirtual())
            MRI->setRegAllocationHint(Src2->getReg(), 0, VCCReg);
          Next = true;
        }

        if (Next)
          continue;
      }

      // Pre-GFX10, shrinking VOP3 instructions pre-RA gave us the chance to
      // fold an immediate into the shrunk instruction as a literal operand. In
      // GFX10 VOP3 instructions can take a literal operand anyway, so there is
      // no advantage to doing this.
      // However, if 64-bit literals are allowed we still need to shrink it
      // for such literal to be able to fold.
      if (ST->hasVOP3Literal() &&
          (!ST->has64BitLiterals() || AMDGPU::isTrue16Inst(MI.getOpcode())) &&
          !IsPostRA)
        continue;

      if (ST->hasTrue16BitInsts() && AMDGPU::isTrue16Inst(MI.getOpcode()) &&
          !shouldShrinkTrue16(MI))
        continue;

      // We can shrink this instruction
      LLVM_DEBUG(dbgs() << "Shrinking " << MI);

      MachineInstr *Inst32 = TII->buildShrunkInst(MI, Op32);
      ++NumInstructionsShrunk;

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `AMDGPU::isTrue16Inst`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`AMDGPU::isTrue16Inst`。

### Lines 1129-1162: Conditional logic and checks
```cpp
      // Copy extra operands not present in the instruction definition.
      copyExtraImplicitOps(*Inst32, MI);

      // Copy deadness from the old explicit vcc def to the new implicit def.
      if (SDst && SDst->isDead())
        Inst32->findRegisterDefOperand(VCCReg, /*TRI=*/nullptr)->setIsDead();

      MI.eraseFromParent();
      foldImmediates(*Inst32);

      LLVM_DEBUG(dbgs() << "e32 MI = " << *Inst32 << '\n');
      Changed = true;
    }
  }
  return Changed;
}

bool SIShrinkInstructionsLegacy::runOnMachineFunction(MachineFunction &MF) {
  if (skipFunction(MF.getFunction()))
    return false;

  return SIShrinkInstructions().run(MF);
}

PreservedAnalyses
SIShrinkInstructionsPass::run(MachineFunction &MF,
                              MachineFunctionAnalysisManager &) {
  if (MF.getFunction().hasOptNone() || !SIShrinkInstructions().run(MF))
    return PreservedAnalyses::all();

  auto PA = getMachineFunctionPassPreservedAnalyses();
  PA.preserveSet<CFGAnalyses>();
  return PA;
}
```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `SIShrinkInstructionsLegacy::runOnMachineFunction`, `SIShrinkInstructionsPass::run`, `PreservedAnalyses::all`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`SIShrinkInstructionsLegacy::runOnMachineFunction`, `SIShrinkInstructionsPass::run`, `PreservedAnalyses::all`。

## Key Concepts / 关键概念
- **Language / 语言**: C++ source
- **Primary symbols / 主要符号**: `ChangeKind`, `SIShrinkInstructions`, `SIShrinkInstructionsLegacy`, `MachineFunctionPass::getAnalysisUsage`, `llvm::createSIShrinkInstructionsLegacyPass`, `SIShrinkInstructions::foldImmediates`
- **Main themes / 核心主题**: register management / 寄存器管理; instruction semantics / 指令语义; subtarget modeling / 子目标建模; legalization / 合法化
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"SIShrinkInstructions.h"`
- `"AMDGPU.h"`
- `"GCNSubtarget.h"`
- `"MCTargetDesc/AMDGPUMCTargetDesc.h"`
- `"Utils/AMDGPUBaseInfo.h"`
- `"llvm/ADT/Statistic.h"`
- `"llvm/CodeGen/MachineFunctionPass.h"`
