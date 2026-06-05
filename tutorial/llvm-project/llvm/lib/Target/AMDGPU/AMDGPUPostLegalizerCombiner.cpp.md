# AMDGPUPostLegalizerCombiner.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/AMDGPUPostLegalizerCombiner.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This source file implements AMDGPUPostLegalizerCombiner for the LLVM AMDGPU backend. It contains target-specific logic used during analysis, lowering, code generation, or pass execution. / 该源文件实现 LLVM AMDGPU 后端中的 AMDGPUPostLegalizerCombiner 相关功能。它包含分析、降低、代码生成或 Pass 执行过程中使用的目标专用逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-34: File banner, includes, and setup
```cpp
//=== lib/CodeGen/GlobalISel/AMDGPUPostLegalizerCombiner.cpp --------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This pass does combining of machine instructions at the generic MI level,
// after the legalizer.
//
//===----------------------------------------------------------------------===//

#include "AMDGPU.h"
#include "AMDGPUCombinerHelper.h"
#include "AMDGPULegalizerInfo.h"
#include "GCNSubtarget.h"
#include "MCTargetDesc/AMDGPUMCTargetDesc.h"
#include "llvm/CodeGen/GlobalISel/Combiner.h"
#include "llvm/CodeGen/GlobalISel/CombinerHelper.h"
#include "llvm/CodeGen/GlobalISel/CombinerInfo.h"
#include "llvm/CodeGen/GlobalISel/GIMatchTableExecutorImpl.h"
#include "llvm/CodeGen/GlobalISel/GISelValueTracking.h"
#include "llvm/CodeGen/GlobalISel/GenericMachineInstrs.h"
#include "llvm/CodeGen/GlobalISel/MIPatternMatch.h"
#include "llvm/CodeGen/MachineDominators.h"
#include "llvm/CodeGen/TargetPassConfig.h"
#include "llvm/IR/IntrinsicsAMDGPU.h"
#include "llvm/Target/TargetMachine.h"

#define GET_GICOMBINER_DEPS
#include "AMDGPUGenPreLegalizeGICombiner.inc"
#undef GET_GICOMBINER_DEPS

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。

### Lines 35-65: Header dependencies and setup
```cpp
#define DEBUG_TYPE "amdgpu-postlegalizer-combiner"

using namespace llvm;
using namespace MIPatternMatch;

namespace {
#define GET_GICOMBINER_TYPES
#include "AMDGPUGenPostLegalizeGICombiner.inc"
#undef GET_GICOMBINER_TYPES

class AMDGPUPostLegalizerCombinerImpl : public Combiner {
protected:
  const AMDGPUPostLegalizerCombinerImplRuleConfig &RuleConfig;
  const GCNSubtarget &STI;
  const SIInstrInfo &TII;
  // TODO: Make CombinerHelper methods const.
  mutable AMDGPUCombinerHelper Helper;

public:
  AMDGPUPostLegalizerCombinerImpl(
      MachineFunction &MF, CombinerInfo &CInfo, GISelValueTracking &VT,
      GISelCSEInfo *CSEInfo,
      const AMDGPUPostLegalizerCombinerImplRuleConfig &RuleConfig,
      const GCNSubtarget &STI, MachineDominatorTree *MDT,
      const LegalizerInfo *LI);

  static const char *getName() { return "AMDGPUPostLegalizerCombinerImpl"; }

  bool tryCombineAllImpl(MachineInstr &I) const;
  bool tryCombineAll(MachineInstr &I) const override;

```
**EN:** These includes pull in the LLVM and AMDGPU declarations required by the rest of the file, making later symbols and helper APIs available. Main symbols: `AMDGPUPostLegalizerCombinerImpl`.
**CN:** 这些头文件引入本文件后续实现所需的 LLVM 与 AMDGPU 声明，使后面的符号和辅助 API 可以直接使用。 主要符号：`AMDGPUPostLegalizerCombinerImpl`。

### Lines 66-99: Declares struct FMinFMaxLegacyInfo
```cpp
  struct FMinFMaxLegacyInfo {
    Register LHS;
    Register RHS;
    CmpInst::Predicate Pred;
  };

  // TODO: Make sure fmin_legacy/fmax_legacy don't canonicalize
  bool matchFMinFMaxLegacy(MachineInstr &MI, MachineInstr &FCmp,
                           FMinFMaxLegacyInfo &Info) const;
  void applySelectFCmpToFMinFMaxLegacy(MachineInstr &MI,
                                       const FMinFMaxLegacyInfo &Info) const;

  bool matchUCharToFloat(MachineInstr &MI) const;
  void applyUCharToFloat(MachineInstr &MI) const;

  bool
  matchRcpSqrtToRsq(MachineInstr &MI,
                    std::function<void(MachineIRBuilder &)> &MatchInfo) const;

  bool matchFDivSqrtToRsqF16(MachineInstr &MI) const;
  void applyFDivSqrtToRsqF16(MachineInstr &MI, const Register &X) const;

  // FIXME: Should be able to have 2 separate matchdatas rather than custom
  // struct boilerplate.
  struct CvtF32UByteMatchInfo {
    Register CvtVal;
    unsigned ShiftOffset;
  };

  bool matchCvtF32UByteN(MachineInstr &MI,
                         CvtF32UByteMatchInfo &MatchInfo) const;
  void applyCvtF32UByteN(MachineInstr &MI,
                         const CvtF32UByteMatchInfo &MatchInfo) const;

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `FMinFMaxLegacyInfo`, `CvtF32UByteMatchInfo`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`FMinFMaxLegacyInfo`, `CvtF32UByteMatchInfo`。

### Lines 100-129: Header dependencies and setup
```cpp
  bool matchRemoveFcanonicalize(MachineInstr &MI, Register &Reg) const;

  // Combine unsigned buffer load and signed extension instructions to generate
  // signed buffer load instructions.
  bool matchCombineSignExtendInReg(
      MachineInstr &MI, std::pair<MachineInstr *, unsigned> &MatchInfo) const;
  void applyCombineSignExtendInReg(
      MachineInstr &MI, std::pair<MachineInstr *, unsigned> &MatchInfo) const;

  // Find the s_mul_u64 instructions where the higher bits are either
  // zero-extended or sign-extended.
  // Replace the s_mul_u64 instructions with S_MUL_I64_I32_PSEUDO if the higher
  // 33 bits are sign extended and with S_MUL_U64_U32_PSEUDO if the higher 32
  // bits are zero extended.
  bool matchCombine_s_mul_u64(MachineInstr &MI, unsigned &NewOpcode) const;

private:
#define GET_GICOMBINER_CLASS_MEMBERS
#define AMDGPUSubtarget GCNSubtarget
#include "AMDGPUGenPostLegalizeGICombiner.inc"
#undef GET_GICOMBINER_CLASS_MEMBERS
#undef AMDGPUSubtarget
};

#define GET_GICOMBINER_IMPL
#define AMDGPUSubtarget GCNSubtarget
#include "AMDGPUGenPostLegalizeGICombiner.inc"
#undef AMDGPUSubtarget
#undef GET_GICOMBINER_IMPL

```
**EN:** These includes pull in the LLVM and AMDGPU declarations required by the rest of the file, making later symbols and helper APIs available.
**CN:** 这些头文件引入本文件后续实现所需的 LLVM 与 AMDGPU 声明，使后面的符号和辅助 API 可以直接使用。

### Lines 130-160: Header dependencies and setup
```cpp
AMDGPUPostLegalizerCombinerImpl::AMDGPUPostLegalizerCombinerImpl(
    MachineFunction &MF, CombinerInfo &CInfo, GISelValueTracking &VT,
    GISelCSEInfo *CSEInfo,
    const AMDGPUPostLegalizerCombinerImplRuleConfig &RuleConfig,
    const GCNSubtarget &STI, MachineDominatorTree *MDT, const LegalizerInfo *LI)
    : Combiner(MF, CInfo, &VT, CSEInfo), RuleConfig(RuleConfig), STI(STI),
      TII(*STI.getInstrInfo()),
      Helper(Observer, B, /*IsPreLegalize*/ false, &VT, MDT, LI, STI),
#define GET_GICOMBINER_CONSTRUCTOR_INITS
#include "AMDGPUGenPostLegalizeGICombiner.inc"
#undef GET_GICOMBINER_CONSTRUCTOR_INITS
{
}

bool AMDGPUPostLegalizerCombinerImpl::tryCombineAll(MachineInstr &MI) const {
  if (tryCombineAllImpl(MI))
    return true;

  switch (MI.getOpcode()) {
  case TargetOpcode::G_SHL:
  case TargetOpcode::G_LSHR:
  case TargetOpcode::G_ASHR:
    // On some subtargets, 64-bit shift is a quarter rate instruction. In the
    // common case, splitting this into a move and a 32-bit shift is faster and
    // the same code size.
    return Helper.tryCombineShiftToUnmerge(MI, 32);
  }

  return false;
}

```
**EN:** These includes pull in the LLVM and AMDGPU declarations required by the rest of the file, making later symbols and helper APIs available. Main symbols: `AMDGPUPostLegalizerCombinerImpl::AMDGPUPostLegalizerCombinerImpl`, `AMDGPUPostLegalizerCombinerImpl::tryCombineAll`.
**CN:** 这些头文件引入本文件后续实现所需的 LLVM 与 AMDGPU 声明，使后面的符号和辅助 API 可以直接使用。 主要符号：`AMDGPUPostLegalizerCombinerImpl::AMDGPUPostLegalizerCombinerImpl`, `AMDGPUPostLegalizerCombinerImpl::tryCombineAll`。

### Lines 161-188: Implements AMDGPUPostLegalizerCombinerImpl::matchFMinFMaxLegacy
```cpp
bool AMDGPUPostLegalizerCombinerImpl::matchFMinFMaxLegacy(
    MachineInstr &MI, MachineInstr &FCmp, FMinFMaxLegacyInfo &Info) const {
  if (!MRI.hasOneNonDBGUse(FCmp.getOperand(0).getReg()))
    return false;

  Info.Pred =
      static_cast<CmpInst::Predicate>(FCmp.getOperand(1).getPredicate());
  Info.LHS = FCmp.getOperand(2).getReg();
  Info.RHS = FCmp.getOperand(3).getReg();
  Register True = MI.getOperand(2).getReg();
  Register False = MI.getOperand(3).getReg();

  // TODO: Handle case where the the selected value is an fneg and the compared
  // constant is the negation of the selected value.
  if ((Info.LHS != True || Info.RHS != False) &&
      (Info.LHS != False || Info.RHS != True))
    return false;

  // Invert the predicate if necessary so that the apply function can assume
  // that the select operands are the same as the fcmp operands.
  // (select (fcmp P, L, R), R, L) -> (select (fcmp !P, L, R), L, R)
  if (Info.LHS != True)
    Info.Pred = CmpInst::getInversePredicate(Info.Pred);

  // Only match </<=/>=/> not ==/!= etc.
  return Info.Pred != CmpInst::getSwappedPredicate(Info.Pred);
}

```
**EN:** This section contains concrete logic for AMDGPUPostLegalizerCombinerImpl::matchFMinFMaxLegacy. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPUPostLegalizerCombinerImpl::matchFMinFMaxLegacy`, `CmpInst::getInversePredicate`, `CmpInst::getSwappedPredicate`.
**CN:** 本节包含与 AMDGPUPostLegalizerCombinerImpl::matchFMinFMaxLegacy 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPUPostLegalizerCombinerImpl::matchFMinFMaxLegacy`, `CmpInst::getInversePredicate`, `CmpInst::getSwappedPredicate`。

### Lines 189-210: Implements AMDGPUPostLegalizerCombinerImpl::applySelectFCmpToFMinFMaxLegacy
```cpp
void AMDGPUPostLegalizerCombinerImpl::applySelectFCmpToFMinFMaxLegacy(
    MachineInstr &MI, const FMinFMaxLegacyInfo &Info) const {
  unsigned Opc = (Info.Pred & CmpInst::FCMP_OGT) ? AMDGPU::G_AMDGPU_FMAX_LEGACY
                                                 : AMDGPU::G_AMDGPU_FMIN_LEGACY;
  Register X = Info.LHS;
  Register Y = Info.RHS;
  if (Info.Pred == CmpInst::getUnorderedPredicate(Info.Pred)) {
    // We need to permute the operands to get the correct NaN behavior. The
    // selected operand is the second one based on the failing compare with NaN,
    // so permute it based on the compare type the hardware uses.
    std::swap(X, Y);
  }

  B.buildInstr(Opc, {MI.getOperand(0)}, {X, Y}, MI.getFlags());

  MI.eraseFromParent();
}

bool AMDGPUPostLegalizerCombinerImpl::matchUCharToFloat(
    MachineInstr &MI) const {
  Register DstReg = MI.getOperand(0).getReg();

```
**EN:** This section contains concrete logic for AMDGPUPostLegalizerCombinerImpl::applySelectFCmpToFMinFMaxLegacy. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPUPostLegalizerCombinerImpl::applySelectFCmpToFMinFMaxLegacy`, `CmpInst::getUnorderedPredicate`, `std::swap`.
**CN:** 本节包含与 AMDGPUPostLegalizerCombinerImpl::applySelectFCmpToFMinFMaxLegacy 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPUPostLegalizerCombinerImpl::applySelectFCmpToFMinFMaxLegacy`, `CmpInst::getUnorderedPredicate`, `std::swap`。

### Lines 211-237: Declares getType
```cpp
  // TODO: We could try to match extracting the higher bytes, which would be
  // easier if i8 vectors weren't promoted to i32 vectors, particularly after
  // types are legalized. v4i8 -> v4f32 is probably the only case to worry
  // about in practice.
  LLT Ty = MRI.getType(DstReg);
  if (Ty == LLT::scalar(32) || Ty == LLT::scalar(16)) {
    Register SrcReg = MI.getOperand(1).getReg();
    unsigned SrcSize = MRI.getType(SrcReg).getSizeInBits();
    assert(SrcSize == 16 || SrcSize == 32 || SrcSize == 64);
    const APInt Mask = APInt::getHighBitsSet(SrcSize, SrcSize - 8);
    return Helper.getValueTracking()->maskedValueIsZero(SrcReg, Mask);
  }

  return false;
}

void AMDGPUPostLegalizerCombinerImpl::applyUCharToFloat(
    MachineInstr &MI) const {
  const LLT S32 = LLT::scalar(32);

  Register DstReg = MI.getOperand(0).getReg();
  Register SrcReg = MI.getOperand(1).getReg();
  LLT Ty = MRI.getType(DstReg);
  LLT SrcTy = MRI.getType(SrcReg);
  if (SrcTy != S32)
    SrcReg = B.buildAnyExtOrTrunc(S32, SrcReg).getReg(0);

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend. Main symbols: `LLT::scalar`, `APInt::getHighBitsSet`, `AMDGPUPostLegalizerCombinerImpl::applyUCharToFloat`.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。 主要符号：`LLT::scalar`, `APInt::getHighBitsSet`, `AMDGPUPostLegalizerCombinerImpl::applyUCharToFloat`。

### Lines 238-263: Declares eraseFromParent
```cpp
  if (Ty == S32) {
    B.buildInstr(AMDGPU::G_AMDGPU_CVT_F32_UBYTE0, {DstReg}, {SrcReg},
                 MI.getFlags());
  } else {
    auto Cvt0 = B.buildInstr(AMDGPU::G_AMDGPU_CVT_F32_UBYTE0, {S32}, {SrcReg},
                             MI.getFlags());
    B.buildFPTrunc(DstReg, Cvt0, MI.getFlags());
  }

  MI.eraseFromParent();
}

bool AMDGPUPostLegalizerCombinerImpl::matchRcpSqrtToRsq(
    MachineInstr &MI,
    std::function<void(MachineIRBuilder &)> &MatchInfo) const {
  auto getRcpSrc = [=](const MachineInstr &MI) -> MachineInstr * {
    if (!MI.getFlag(MachineInstr::FmContract))
      return nullptr;

    if (auto *GI = dyn_cast<GIntrinsic>(&MI)) {
      if (GI->is(Intrinsic::amdgcn_rcp))
        return MRI.getVRegDef(MI.getOperand(2).getReg());
    }
    return nullptr;
  };

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend. Main symbols: `AMDGPUPostLegalizerCombinerImpl::matchRcpSqrtToRsq`.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。 主要符号：`AMDGPUPostLegalizerCombinerImpl::matchRcpSqrtToRsq`。

### Lines 264-296: Conditional logic and checks
```cpp
  auto getSqrtSrc = [=](const MachineInstr &MI) -> MachineInstr * {
    if (!MI.getFlag(MachineInstr::FmContract))
      return nullptr;
    MachineInstr *SqrtSrcMI = nullptr;
    auto Match =
        mi_match(MI.getOperand(0).getReg(), MRI, m_GFSqrt(m_MInstr(SqrtSrcMI)));
    (void)Match;
    return SqrtSrcMI;
  };

  MachineInstr *RcpSrcMI = nullptr, *SqrtSrcMI = nullptr;
  // rcp(sqrt(x))
  if ((RcpSrcMI = getRcpSrc(MI)) && (SqrtSrcMI = getSqrtSrc(*RcpSrcMI))) {
    MatchInfo = [SqrtSrcMI, &MI](MachineIRBuilder &B) {
      B.buildIntrinsic(Intrinsic::amdgcn_rsq, {MI.getOperand(0)})
          .addUse(SqrtSrcMI->getOperand(0).getReg())
          .setMIFlags(MI.getFlags());
    };
    return true;
  }

  // sqrt(rcp(x))
  if ((SqrtSrcMI = getSqrtSrc(MI)) && (RcpSrcMI = getRcpSrc(*SqrtSrcMI))) {
    MatchInfo = [RcpSrcMI, &MI](MachineIRBuilder &B) {
      B.buildIntrinsic(Intrinsic::amdgcn_rsq, {MI.getOperand(0)})
          .addUse(RcpSrcMI->getOperand(0).getReg())
          .setMIFlags(MI.getFlags());
    };
    return true;
  }
  return false;
}

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 297-329: Implements AMDGPUPostLegalizerCombinerImpl::matchFDivSqrtToRsqF16
```cpp
bool AMDGPUPostLegalizerCombinerImpl::matchFDivSqrtToRsqF16(
    MachineInstr &MI) const {
  Register Sqrt = MI.getOperand(2).getReg();
  return MRI.hasOneNonDBGUse(Sqrt);
}

void AMDGPUPostLegalizerCombinerImpl::applyFDivSqrtToRsqF16(
    MachineInstr &MI, const Register &X) const {
  Register Dst = MI.getOperand(0).getReg();
  Register Y = MI.getOperand(1).getReg();
  LLT DstTy = MRI.getType(Dst);
  uint32_t Flags = MI.getFlags();
  Register RSQ = B.buildIntrinsic(Intrinsic::amdgcn_rsq, {DstTy})
                     .addUse(X)
                     .setMIFlags(Flags)
                     .getReg(0);
  B.buildFMul(Dst, RSQ, Y, Flags);
  MI.eraseFromParent();
}

bool AMDGPUPostLegalizerCombinerImpl::matchCvtF32UByteN(
    MachineInstr &MI, CvtF32UByteMatchInfo &MatchInfo) const {
  Register SrcReg = MI.getOperand(1).getReg();

  // Look through G_ZEXT.
  bool IsShr = mi_match(SrcReg, MRI, m_GZExt(m_Reg(SrcReg)));

  Register Src0;
  int64_t ShiftAmt;
  IsShr = mi_match(SrcReg, MRI, m_GLShr(m_Reg(Src0), m_ICst(ShiftAmt)));
  if (IsShr || mi_match(SrcReg, MRI, m_GShl(m_Reg(Src0), m_ICst(ShiftAmt)))) {
    const unsigned Offset = MI.getOpcode() - AMDGPU::G_AMDGPU_CVT_F32_UBYTE0;

```
**EN:** This section contains concrete logic for AMDGPUPostLegalizerCombinerImpl::matchFDivSqrtToRsqF16. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPUPostLegalizerCombinerImpl::matchFDivSqrtToRsqF16`, `AMDGPUPostLegalizerCombinerImpl::applyFDivSqrtToRsqF16`, `AMDGPUPostLegalizerCombinerImpl::matchCvtF32UByteN`.
**CN:** 本节包含与 AMDGPUPostLegalizerCombinerImpl::matchFDivSqrtToRsqF16 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPUPostLegalizerCombinerImpl::matchFDivSqrtToRsqF16`, `AMDGPUPostLegalizerCombinerImpl::applyFDivSqrtToRsqF16`, `AMDGPUPostLegalizerCombinerImpl::matchCvtF32UByteN`。

### Lines 330-361: Conditional logic and checks
```cpp
    unsigned ShiftOffset = 8 * Offset;
    if (IsShr)
      ShiftOffset += ShiftAmt;
    else
      ShiftOffset -= ShiftAmt;

    MatchInfo.CvtVal = Src0;
    MatchInfo.ShiftOffset = ShiftOffset;
    return ShiftOffset < 32 && ShiftOffset >= 8 && (ShiftOffset % 8) == 0;
  }

  // TODO: Simplify demanded bits.
  return false;
}

void AMDGPUPostLegalizerCombinerImpl::applyCvtF32UByteN(
    MachineInstr &MI, const CvtF32UByteMatchInfo &MatchInfo) const {
  unsigned NewOpc = AMDGPU::G_AMDGPU_CVT_F32_UBYTE0 + MatchInfo.ShiftOffset / 8;

  const LLT S32 = LLT::scalar(32);
  Register CvtSrc = MatchInfo.CvtVal;
  LLT SrcTy = MRI.getType(MatchInfo.CvtVal);
  if (SrcTy != S32) {
    assert(SrcTy.isScalar() && SrcTy.getSizeInBits() >= 8);
    CvtSrc = B.buildAnyExt(S32, CvtSrc).getReg(0);
  }

  assert(MI.getOpcode() != NewOpc);
  B.buildInstr(NewOpc, {MI.getOperand(0)}, {CvtSrc}, MI.getFlags());
  MI.eraseFromParent();
}

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `AMDGPUPostLegalizerCombinerImpl::applyCvtF32UByteN`, `LLT::scalar`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`AMDGPUPostLegalizerCombinerImpl::applyCvtF32UByteN`, `LLT::scalar`。

### Lines 362-381: Implements AMDGPUPostLegalizerCombinerImpl::matchRemoveFcanonicalize
```cpp
bool AMDGPUPostLegalizerCombinerImpl::matchRemoveFcanonicalize(
    MachineInstr &MI, Register &Reg) const {
  const SITargetLowering *TLI = static_cast<const SITargetLowering *>(
      MF.getSubtarget().getTargetLowering());
  Reg = MI.getOperand(1).getReg();
  return TLI->isCanonicalized(Reg, MF);
}

// The buffer_load_{i8, i16} intrinsics are initially lowered as
// buffer_load_{u8, u16} instructions. Here, the buffer_load_{u8, u16}
// instructions are combined with sign extension instrucions in order to
// generate buffer_load_{i8, i16} instructions.

// Identify buffer_load_{u8, u16}.
bool AMDGPUPostLegalizerCombinerImpl::matchCombineSignExtendInReg(
    MachineInstr &MI, std::pair<MachineInstr *, unsigned> &MatchData) const {
  Register LoadReg = MI.getOperand(1).getReg();
  if (!MRI.hasOneNonDBGUse(LoadReg))
    return false;

```
**EN:** This section contains concrete logic for AMDGPUPostLegalizerCombinerImpl::matchRemoveFcanonicalize. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPUPostLegalizerCombinerImpl::matchRemoveFcanonicalize`, `AMDGPUPostLegalizerCombinerImpl::matchCombineSignExtendInReg`.
**CN:** 本节包含与 AMDGPUPostLegalizerCombinerImpl::matchRemoveFcanonicalize 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPUPostLegalizerCombinerImpl::matchRemoveFcanonicalize`, `AMDGPUPostLegalizerCombinerImpl::matchCombineSignExtendInReg`。

### Lines 382-402: Declares getVRegDef
```cpp
  // Check if the first operand of the sign extension is a subword buffer load
  // instruction.
  MachineInstr *LoadMI = MRI.getVRegDef(LoadReg);
  int64_t Width = MI.getOperand(2).getImm();
  switch (LoadMI->getOpcode()) {
  case AMDGPU::G_AMDGPU_BUFFER_LOAD_UBYTE:
    MatchData = {LoadMI, AMDGPU::G_AMDGPU_BUFFER_LOAD_SBYTE};
    return Width == 8;
  case AMDGPU::G_AMDGPU_BUFFER_LOAD_USHORT:
    MatchData = {LoadMI, AMDGPU::G_AMDGPU_BUFFER_LOAD_SSHORT};
    return Width == 16;
  case AMDGPU::G_AMDGPU_S_BUFFER_LOAD_UBYTE:
    MatchData = {LoadMI, AMDGPU::G_AMDGPU_S_BUFFER_LOAD_SBYTE};
    return Width == 8;
  case AMDGPU::G_AMDGPU_S_BUFFER_LOAD_USHORT:
    MatchData = {LoadMI, AMDGPU::G_AMDGPU_S_BUFFER_LOAD_SSHORT};
    return Width == 16;
  }
  return false;
}

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。

### Lines 403-429: Implements AMDGPUPostLegalizerCombinerImpl::applyCombineSignExtendInReg
```cpp
// Combine buffer_load_{u8, u16} and the sign extension instruction to generate
// buffer_load_{i8, i16}.
void AMDGPUPostLegalizerCombinerImpl::applyCombineSignExtendInReg(
    MachineInstr &MI, std::pair<MachineInstr *, unsigned> &MatchData) const {
  auto [LoadMI, NewOpcode] = MatchData;
  LoadMI->setDesc(TII.get(NewOpcode));
  // Update the destination register of the load with the destination register
  // of the sign extension.
  Register SignExtendInsnDst = MI.getOperand(0).getReg();
  LoadMI->getOperand(0).setReg(SignExtendInsnDst);
  // Remove the sign extension.
  MI.eraseFromParent();
}

bool AMDGPUPostLegalizerCombinerImpl::matchCombine_s_mul_u64(
    MachineInstr &MI, unsigned &NewOpcode) const {
  Register Src0 = MI.getOperand(1).getReg();
  Register Src1 = MI.getOperand(2).getReg();
  if (MRI.getType(Src0) != LLT::scalar(64))
    return false;

  if (VT->getKnownBits(Src1).countMinLeadingZeros() >= 32 &&
      VT->getKnownBits(Src0).countMinLeadingZeros() >= 32) {
    NewOpcode = AMDGPU::G_AMDGPU_S_MUL_U64_U32;
    return true;
  }

```
**EN:** This section contains concrete logic for AMDGPUPostLegalizerCombinerImpl::applyCombineSignExtendInReg. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPUPostLegalizerCombinerImpl::applyCombineSignExtendInReg`, `AMDGPUPostLegalizerCombinerImpl::matchCombine_s_mul_u64`, `LLT::scalar`.
**CN:** 本节包含与 AMDGPUPostLegalizerCombinerImpl::applyCombineSignExtendInReg 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPUPostLegalizerCombinerImpl::applyCombineSignExtendInReg`, `AMDGPUPostLegalizerCombinerImpl::matchCombine_s_mul_u64`, `LLT::scalar`。

### Lines 430-460: Declares class AMDGPUPostLegalizerCombiner
```cpp
  if (VT->computeNumSignBits(Src1) >= 33 &&
      VT->computeNumSignBits(Src0) >= 33) {
    NewOpcode = AMDGPU::G_AMDGPU_S_MUL_I64_I32;
    return true;
  }
  return false;
}

// Pass boilerplate
// ================

class AMDGPUPostLegalizerCombiner : public MachineFunctionPass {
public:
  static char ID;

  AMDGPUPostLegalizerCombiner(bool IsOptNone = false);

  StringRef getPassName() const override {
    return "AMDGPUPostLegalizerCombiner";
  }

  bool runOnMachineFunction(MachineFunction &MF) override;

  void getAnalysisUsage(AnalysisUsage &AU) const override;

private:
  bool IsOptNone;
  AMDGPUPostLegalizerCombinerImplRuleConfig RuleConfig;
};
} // end anonymous namespace

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `AMDGPUPostLegalizerCombiner`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`AMDGPUPostLegalizerCombiner`。

### Lines 461-489: Implements AMDGPUPostLegalizerCombiner::getAnalysisUsage
```cpp
void AMDGPUPostLegalizerCombiner::getAnalysisUsage(AnalysisUsage &AU) const {
  AU.setPreservesCFG();
  getSelectionDAGFallbackAnalysisUsage(AU);
  AU.addRequired<GISelValueTrackingAnalysisLegacy>();
  AU.addPreserved<GISelValueTrackingAnalysisLegacy>();
  if (!IsOptNone) {
    AU.addRequired<MachineDominatorTreeWrapperPass>();
    AU.addPreserved<MachineDominatorTreeWrapperPass>();
  }
  MachineFunctionPass::getAnalysisUsage(AU);
}

AMDGPUPostLegalizerCombiner::AMDGPUPostLegalizerCombiner(bool IsOptNone)
    : MachineFunctionPass(ID), IsOptNone(IsOptNone) {
  if (!RuleConfig.parseCommandLineOption())
    report_fatal_error("Invalid rule identifier");
}

bool AMDGPUPostLegalizerCombiner::runOnMachineFunction(MachineFunction &MF) {
  if (MF.getProperties().hasFailedISel())
    return false;
  const Function &F = MF.getFunction();
  bool EnableOpt =
      MF.getTarget().getOptLevel() != CodeGenOptLevel::None && !skipFunction(F);

  const GCNSubtarget &ST = MF.getSubtarget<GCNSubtarget>();
  const AMDGPULegalizerInfo *LI =
      static_cast<const AMDGPULegalizerInfo *>(ST.getLegalizerInfo());

```
**EN:** This section contains concrete logic for AMDGPUPostLegalizerCombiner::getAnalysisUsage. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPUPostLegalizerCombiner::getAnalysisUsage`, `MachineFunctionPass::getAnalysisUsage`, `AMDGPUPostLegalizerCombiner::AMDGPUPostLegalizerCombiner`.
**CN:** 本节包含与 AMDGPUPostLegalizerCombiner::getAnalysisUsage 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPUPostLegalizerCombiner::getAnalysisUsage`, `MachineFunctionPass::getAnalysisUsage`, `AMDGPUPostLegalizerCombiner::AMDGPUPostLegalizerCombiner`。

### Lines 490-519: Registers LLVM passes
```cpp
  GISelValueTracking *VT =
      &getAnalysis<GISelValueTrackingAnalysisLegacy>().get(MF);
  MachineDominatorTree *MDT =
      IsOptNone ? nullptr
                : &getAnalysis<MachineDominatorTreeWrapperPass>().getDomTree();

  CombinerInfo CInfo(/*AllowIllegalOps*/ false, /*ShouldLegalizeIllegal*/ true,
                     LI, EnableOpt, F.hasOptSize(), F.hasMinSize());
  // Disable fixed-point iteration to reduce compile-time
  CInfo.MaxIterations = 1;
  CInfo.ObserverLvl = CombinerInfo::ObserverLevel::SinglePass;
  // Legalizer performs DCE, so a full DCE pass is unnecessary.
  CInfo.EnableFullDCE = false;
  AMDGPUPostLegalizerCombinerImpl Impl(MF, CInfo, *VT, /*CSEInfo*/ nullptr,
                                       RuleConfig, ST, MDT, LI);
  return Impl.combineMachineInstrs();
}

char AMDGPUPostLegalizerCombiner::ID = 0;
INITIALIZE_PASS_BEGIN(AMDGPUPostLegalizerCombiner, DEBUG_TYPE,
                      "Combine AMDGPU machine instrs after legalization", false,
                      false)
INITIALIZE_PASS_DEPENDENCY(GISelValueTrackingAnalysisLegacy)
INITIALIZE_PASS_END(AMDGPUPostLegalizerCombiner, DEBUG_TYPE,
                    "Combine AMDGPU machine instrs after legalization", false,
                    false)

FunctionPass *llvm::createAMDGPUPostLegalizeCombiner(bool IsOptNone) {
  return new AMDGPUPostLegalizerCombiner(IsOptNone);
}
```
**EN:** This section connects the implementation to LLVM pass registration so the AMDGPU-specific analysis or transformation can be constructed by the pass framework. Main symbols: `llvm::createAMDGPUPostLegalizeCombiner`.
**CN:** 本节把实现接入 LLVM 的 Pass 注册机制，使 AMDGPU 专用分析或变换能够被 Pass 框架创建和调用。 主要符号：`llvm::createAMDGPUPostLegalizeCombiner`。

## Key Concepts / 关键概念
- **Language / 语言**: C++ source
- **Primary symbols / 主要符号**: `AMDGPUPostLegalizerCombinerImpl`, `FMinFMaxLegacyInfo`, `CvtF32UByteMatchInfo`, `AMDGPUPostLegalizerCombiner`, `AMDGPUPostLegalizerCombinerImpl::AMDGPUPostLegalizerCombinerImpl`, `AMDGPUPostLegalizerCombinerImpl::tryCombineAll`
- **Main themes / 核心主题**: register management / 寄存器管理; instruction semantics / 指令语义; subtarget modeling / 子目标建模; legalization / 合法化
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"AMDGPU.h"`
- `"AMDGPUCombinerHelper.h"`
- `"AMDGPULegalizerInfo.h"`
- `"GCNSubtarget.h"`
- `"MCTargetDesc/AMDGPUMCTargetDesc.h"`
- `"llvm/CodeGen/GlobalISel/Combiner.h"`
- `"llvm/CodeGen/GlobalISel/CombinerHelper.h"`
- `"llvm/CodeGen/GlobalISel/CombinerInfo.h"`
- `"llvm/CodeGen/GlobalISel/GIMatchTableExecutorImpl.h"`
- `"llvm/CodeGen/GlobalISel/GISelValueTracking.h"`
- `"llvm/CodeGen/GlobalISel/GenericMachineInstrs.h"`
- `"llvm/CodeGen/GlobalISel/MIPatternMatch.h"`
- `"llvm/CodeGen/MachineDominators.h"`
- `"llvm/CodeGen/TargetPassConfig.h"`
- `"llvm/IR/IntrinsicsAMDGPU.h"`
- `"llvm/Target/TargetMachine.h"`
- `"AMDGPUGenPreLegalizeGICombiner.inc"`
- `"AMDGPUGenPostLegalizeGICombiner.inc"`
