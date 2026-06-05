# AMDGPURegBankCombiner.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/AMDGPURegBankCombiner.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This source file implements AMDGPURegBankCombiner for the LLVM AMDGPU backend. It contains target-specific logic used during analysis, lowering, code generation, or pass execution. / 该源文件实现 LLVM AMDGPU 后端中的 AMDGPURegBankCombiner 相关功能。它包含分析、降低、代码生成或 Pass 执行过程中使用的目标专用逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-33: File banner, includes, and setup
```cpp
//=== lib/CodeGen/GlobalISel/AMDGPURegBankCombiner.cpp ---------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This pass does combining of machine instructions at the generic MI level,
// after register banks are known.
//
//===----------------------------------------------------------------------===//

#include "AMDGPU.h"
#include "AMDGPULegalizerInfo.h"
#include "AMDGPURegisterBankInfo.h"
#include "GCNSubtarget.h"
#include "MCTargetDesc/AMDGPUMCTargetDesc.h"
#include "SIMachineFunctionInfo.h"
#include "llvm/CodeGen/GlobalISel/Combiner.h"
#include "llvm/CodeGen/GlobalISel/CombinerHelper.h"
#include "llvm/CodeGen/GlobalISel/CombinerInfo.h"
#include "llvm/CodeGen/GlobalISel/GIMatchTableExecutorImpl.h"
#include "llvm/CodeGen/GlobalISel/GISelValueTracking.h"
#include "llvm/CodeGen/GlobalISel/MIPatternMatch.h"
#include "llvm/CodeGen/MachineDominators.h"
#include "llvm/CodeGen/TargetPassConfig.h"
#include "llvm/Target/TargetMachine.h"

#define GET_GICOMBINER_DEPS
#include "AMDGPUGenPreLegalizeGICombiner.inc"
#undef GET_GICOMBINER_DEPS

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。

### Lines 34-67: Header dependencies and setup
```cpp
#define DEBUG_TYPE "amdgpu-regbank-combiner"

using namespace llvm;
using namespace MIPatternMatch;

namespace {
#define GET_GICOMBINER_TYPES
#include "AMDGPUGenRegBankGICombiner.inc"
#undef GET_GICOMBINER_TYPES

class AMDGPURegBankCombinerImpl : public Combiner {
protected:
  const AMDGPURegBankCombinerImplRuleConfig &RuleConfig;
  const GCNSubtarget &STI;
  const RegisterBankInfo &RBI;
  const TargetRegisterInfo &TRI;
  const SIInstrInfo &TII;
  const CombinerHelper Helper;

public:
  AMDGPURegBankCombinerImpl(
      MachineFunction &MF, CombinerInfo &CInfo, GISelValueTracking &VT,
      GISelCSEInfo *CSEInfo,
      const AMDGPURegBankCombinerImplRuleConfig &RuleConfig,
      const GCNSubtarget &STI, MachineDominatorTree *MDT,
      const LegalizerInfo *LI);

  static const char *getName() { return "AMDGPURegBankCombinerImpl"; }

  bool tryCombineAll(MachineInstr &I) const override;

  bool isVgprRegBank(Register Reg) const;
  Register getAsVgpr(Register Reg) const;

```
**EN:** These includes pull in the LLVM and AMDGPU declarations required by the rest of the file, making later symbols and helper APIs available. Main symbols: `AMDGPURegBankCombinerImpl`.
**CN:** 这些头文件引入本文件后续实现所需的 LLVM 与 AMDGPU 声明，使后面的符号和辅助 API 可以直接使用。 主要符号：`AMDGPURegBankCombinerImpl`。

### Lines 68-95: Declares struct MinMaxMedOpc
```cpp
  struct MinMaxMedOpc {
    unsigned Min, Max, Med;
  };

  struct Med3MatchInfo {
    unsigned Opc;
    Register Val0, Val1, Val2;
  };

  MinMaxMedOpc getMinMaxPair(unsigned Opc) const;

  template <class m_Cst, typename CstTy>
  bool matchMed(MachineInstr &MI, MachineRegisterInfo &MRI, MinMaxMedOpc MMMOpc,
                Register &Val, CstTy &K0, CstTy &K1) const;

  bool matchIntMinMaxToMed3(MachineInstr &MI, Med3MatchInfo &MatchInfo) const;
  bool matchFPMinMaxToMed3(MachineInstr &MI, Med3MatchInfo &MatchInfo) const;
  bool matchFPMinMaxToClamp(MachineInstr &MI, Register &Reg) const;
  bool matchFPMed3ToClamp(MachineInstr &MI, Register &Reg) const;
  void applyMed3(MachineInstr &MI, Med3MatchInfo &MatchInfo) const;
  void applyClamp(MachineInstr &MI, Register &Reg) const;

  void applyCanonicalizeZextShiftAmt(MachineInstr &MI, MachineInstr &Ext) const;

  bool combineD16Load(MachineInstr &MI) const;
  bool applyD16Load(unsigned D16Opc, MachineInstr &DstMI,
                    MachineInstr *SmallLoad, Register ToOverwriteD16) const;

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `MinMaxMedOpc`, `Med3MatchInfo`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`MinMaxMedOpc`, `Med3MatchInfo`。

### Lines 96-116: Header dependencies and setup
```cpp
private:
  SIModeRegisterDefaults getMode() const;
  bool getIEEE() const;
  bool getDX10Clamp() const;
  bool isFminnumIeee(const MachineInstr &MI) const;
  bool isFCst(MachineInstr *MI) const;
  bool isClampZeroToOne(MachineInstr *K0, MachineInstr *K1) const;

#define GET_GICOMBINER_CLASS_MEMBERS
#define AMDGPUSubtarget GCNSubtarget
#include "AMDGPUGenRegBankGICombiner.inc"
#undef GET_GICOMBINER_CLASS_MEMBERS
#undef AMDGPUSubtarget
};

#define GET_GICOMBINER_IMPL
#define AMDGPUSubtarget GCNSubtarget
#include "AMDGPUGenRegBankGICombiner.inc"
#undef AMDGPUSubtarget
#undef GET_GICOMBINER_IMPL

```
**EN:** These includes pull in the LLVM and AMDGPU declarations required by the rest of the file, making later symbols and helper APIs available.
**CN:** 这些头文件引入本文件后续实现所需的 LLVM 与 AMDGPU 声明，使后面的符号和辅助 API 可以直接使用。

### Lines 117-146: Header dependencies and setup
```cpp
AMDGPURegBankCombinerImpl::AMDGPURegBankCombinerImpl(
    MachineFunction &MF, CombinerInfo &CInfo, GISelValueTracking &VT,
    GISelCSEInfo *CSEInfo,
    const AMDGPURegBankCombinerImplRuleConfig &RuleConfig,
    const GCNSubtarget &STI, MachineDominatorTree *MDT, const LegalizerInfo *LI)
    : Combiner(MF, CInfo, &VT, CSEInfo), RuleConfig(RuleConfig), STI(STI),
      RBI(*STI.getRegBankInfo()), TRI(*STI.getRegisterInfo()),
      TII(*STI.getInstrInfo()),
      Helper(Observer, B, /*IsPreLegalize*/ false, &VT, MDT, LI),
#define GET_GICOMBINER_CONSTRUCTOR_INITS
#include "AMDGPUGenRegBankGICombiner.inc"
#undef GET_GICOMBINER_CONSTRUCTOR_INITS
{
}

bool AMDGPURegBankCombinerImpl::isVgprRegBank(Register Reg) const {
  return RBI.getRegBank(Reg, MRI, TRI)->getID() == AMDGPU::VGPRRegBankID;
}

Register AMDGPURegBankCombinerImpl::getAsVgpr(Register Reg) const {
  if (isVgprRegBank(Reg))
    return Reg;

  // Search for existing copy of Reg to vgpr.
  for (MachineInstr &Use : MRI.use_instructions(Reg)) {
    Register Def = Use.getOperand(0).getReg();
    if (Use.getOpcode() == AMDGPU::COPY && isVgprRegBank(Def))
      return Def;
  }

```
**EN:** These includes pull in the LLVM and AMDGPU declarations required by the rest of the file, making later symbols and helper APIs available. Main symbols: `AMDGPURegBankCombinerImpl::AMDGPURegBankCombinerImpl`, `AMDGPURegBankCombinerImpl::isVgprRegBank`, `AMDGPURegBankCombinerImpl::getAsVgpr`.
**CN:** 这些头文件引入本文件后续实现所需的 LLVM 与 AMDGPU 声明，使后面的符号和辅助 API 可以直接使用。 主要符号：`AMDGPURegBankCombinerImpl::AMDGPURegBankCombinerImpl`, `AMDGPURegBankCombinerImpl::isVgprRegBank`, `AMDGPURegBankCombinerImpl::getAsVgpr`。

### Lines 147-173: Declares buildCopy
```cpp
  // Copy Reg to vgpr.
  Register VgprReg = B.buildCopy(MRI.getType(Reg), Reg).getReg(0);
  MRI.setRegBank(VgprReg, RBI.getRegBank(AMDGPU::VGPRRegBankID));
  return VgprReg;
}

AMDGPURegBankCombinerImpl::MinMaxMedOpc
AMDGPURegBankCombinerImpl::getMinMaxPair(unsigned Opc) const {
  switch (Opc) {
  default:
    llvm_unreachable("Unsupported opcode");
  case AMDGPU::G_SMAX:
  case AMDGPU::G_SMIN:
    return {AMDGPU::G_SMIN, AMDGPU::G_SMAX, AMDGPU::G_AMDGPU_SMED3};
  case AMDGPU::G_UMAX:
  case AMDGPU::G_UMIN:
    return {AMDGPU::G_UMIN, AMDGPU::G_UMAX, AMDGPU::G_AMDGPU_UMED3};
  case AMDGPU::G_FMAXNUM:
  case AMDGPU::G_FMINNUM:
    return {AMDGPU::G_FMINNUM, AMDGPU::G_FMAXNUM, AMDGPU::G_AMDGPU_FMED3};
  case AMDGPU::G_FMAXNUM_IEEE:
  case AMDGPU::G_FMINNUM_IEEE:
    return {AMDGPU::G_FMINNUM_IEEE, AMDGPU::G_FMAXNUM_IEEE,
            AMDGPU::G_AMDGPU_FMED3};
  }
}

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend. Main symbols: `AMDGPURegBankCombinerImpl::getMinMaxPair`.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。 主要符号：`AMDGPURegBankCombinerImpl::getMinMaxPair`。

### Lines 174-206: Implements AMDGPURegBankCombinerImpl::matchMed
```cpp
template <class m_Cst, typename CstTy>
bool AMDGPURegBankCombinerImpl::matchMed(MachineInstr &MI,
                                         MachineRegisterInfo &MRI,
                                         MinMaxMedOpc MMMOpc, Register &Val,
                                         CstTy &K0, CstTy &K1) const {
  // 4 operand commutes of: min(max(Val, K0), K1).
  // Find K1 from outer instr: min(max(...), K1) or min(K1, max(...)).
  // Find K0 and Val from inner instr: max(K0, Val) or max(Val, K0).
  // 4 operand commutes of: max(min(Val, K1), K0).
  // Find K0 from outer instr: max(min(...), K0) or max(K0, min(...)).
  // Find K1 and Val from inner instr: min(K1, Val) or min(Val, K1).
  return mi_match(
      MI, MRI,
      m_any_of(
          m_CommutativeBinOp(
              MMMOpc.Min, m_CommutativeBinOp(MMMOpc.Max, m_Reg(Val), m_Cst(K0)),
              m_Cst(K1)),
          m_CommutativeBinOp(
              MMMOpc.Max, m_CommutativeBinOp(MMMOpc.Min, m_Reg(Val), m_Cst(K1)),
              m_Cst(K0))));
}

bool AMDGPURegBankCombinerImpl::matchIntMinMaxToMed3(
    MachineInstr &MI, Med3MatchInfo &MatchInfo) const {
  Register Dst = MI.getOperand(0).getReg();
  if (!isVgprRegBank(Dst))
    return false;

  // med3 for i16 is only available on gfx9+, and not available for v2i16.
  LLT Ty = MRI.getType(Dst);
  if ((Ty != LLT::scalar(16) || !STI.hasMed3_16()) && Ty != LLT::scalar(32))
    return false;

```
**EN:** This section contains concrete logic for AMDGPURegBankCombinerImpl::matchMed. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPURegBankCombinerImpl::matchMed`, `AMDGPURegBankCombinerImpl::matchIntMinMaxToMed3`, `LLT::scalar`.
**CN:** 本节包含与 AMDGPURegBankCombinerImpl::matchMed 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPURegBankCombinerImpl::matchMed`, `AMDGPURegBankCombinerImpl::matchIntMinMaxToMed3`, `LLT::scalar`。

### Lines 207-230: Declares getMinMaxPair
```cpp
  MinMaxMedOpc OpcodeTriple = getMinMaxPair(MI.getOpcode());
  Register Val;
  std::optional<ValueAndVReg> K0, K1;
  // Match min(max(Val, K0), K1) or max(min(Val, K1), K0). Then see if K0 <= K1.
  if (!matchMed<GCstAndRegMatch>(MI, MRI, OpcodeTriple, Val, K0, K1))
    return false;

  if (OpcodeTriple.Med == AMDGPU::G_AMDGPU_SMED3 && K0->Value.sgt(K1->Value))
    return false;
  if (OpcodeTriple.Med == AMDGPU::G_AMDGPU_UMED3 && K0->Value.ugt(K1->Value))
    return false;

  MatchInfo = {OpcodeTriple.Med, Val, K0->VReg, K1->VReg};
  return true;
}

// fmed3(NaN, K0, K1) = min(min(NaN, K0), K1)
// ieee = true  : min/max(SNaN, K) = QNaN, min/max(QNaN, K) = K
// ieee = false : min/max(NaN, K) = K
// clamp(NaN) = dx10_clamp ? 0.0 : NaN
// Consider values of min(max(Val, K0), K1) and max(min(Val, K1), K0) as input.
// Other operand commutes (see matchMed) give same result since min and max are
// commutative.

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。

### Lines 231-260: Implements AMDGPURegBankCombinerImpl::matchFPMinMaxToMed3
```cpp
// Try to replace fp min(max(Val, K0), K1) or max(min(Val, K1), K0), KO<=K1
// with fmed3(Val, K0, K1) or clamp(Val). Clamp requires K0 = 0.0 and K1 = 1.0.
// Val = SNaN only for ieee = true
// fmed3(SNaN, K0, K1) = min(min(SNaN, K0), K1) = min(QNaN, K1) = K1
// min(max(SNaN, K0), K1) = min(QNaN, K1) = K1
// max(min(SNaN, K1), K0) = max(K1, K0) = K1
// Val = NaN,ieee = false or Val = QNaN,ieee = true
// fmed3(NaN, K0, K1) = min(min(NaN, K0), K1) = min(K0, K1) = K0
// min(max(NaN, K0), K1) = min(K0, K1) = K0 (can clamp when dx10_clamp = true)
// max(min(NaN, K1), K0) = max(K1, K0) = K1 != K0
bool AMDGPURegBankCombinerImpl::matchFPMinMaxToMed3(
    MachineInstr &MI, Med3MatchInfo &MatchInfo) const {
  Register Dst = MI.getOperand(0).getReg();
  LLT Ty = MRI.getType(Dst);

  // med3 for f16 is only available on gfx9+, and not available for v2f16.
  if ((Ty != LLT::scalar(16) || !STI.hasMed3_16()) && Ty != LLT::scalar(32))
    return false;

  auto OpcodeTriple = getMinMaxPair(MI.getOpcode());

  Register Val;
  std::optional<FPValueAndVReg> K0, K1;
  // Match min(max(Val, K0), K1) or max(min(Val, K1), K0). Then see if K0 <= K1.
  if (!matchMed<GFCstAndRegMatch>(MI, MRI, OpcodeTriple, Val, K0, K1))
    return false;

  if (K0->Value > K1->Value)
    return false;

```
**EN:** This section contains concrete logic for AMDGPURegBankCombinerImpl::matchFPMinMaxToMed3. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPURegBankCombinerImpl::matchFPMinMaxToMed3`, `LLT::scalar`.
**CN:** 本节包含与 AMDGPURegBankCombinerImpl::matchFPMinMaxToMed3 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPURegBankCombinerImpl::matchFPMinMaxToMed3`, `LLT::scalar`。

### Lines 261-292: Conditional logic and checks
```cpp
  // For IEEE=false perform combine only when it's safe to assume that there are
  // no NaN inputs. Most often MI is marked with nnan fast math flag.
  // For IEEE=true consider NaN inputs. fmed3(NaN, K0, K1) is equivalent to
  // min(min(NaN, K0), K1). Safe to fold for min(max(Val, K0), K1) since inner
  // nodes(max/min) have same behavior when one input is NaN and other isn't.
  // Don't consider max(min(SNaN, K1), K0) since there is no isKnownNeverQNaN,
  // also post-legalizer inputs to min/max are fcanonicalized (never SNaN).
  if ((getIEEE() && isFminnumIeee(MI)) || VT->isKnownNeverNaN(Dst)) {
    // Don't fold single use constant that can't be inlined.
    if ((!MRI.hasOneNonDBGUse(K0->VReg) || TII.isInlineConstant(K0->Value)) &&
        (!MRI.hasOneNonDBGUse(K1->VReg) || TII.isInlineConstant(K1->Value))) {
      MatchInfo = {OpcodeTriple.Med, Val, K0->VReg, K1->VReg};
      return true;
    }
  }

  return false;
}

bool AMDGPURegBankCombinerImpl::matchFPMinMaxToClamp(MachineInstr &MI,
                                                     Register &Reg) const {
  // Clamp is available on all types after regbankselect (f16, f32, f64, v2f16).
  auto OpcodeTriple = getMinMaxPair(MI.getOpcode());
  Register Val;
  std::optional<FPValueAndVReg> K0, K1;
  // Match min(max(Val, K0), K1) or max(min(Val, K1), K0).
  if (!matchMed<GFCstOrSplatGFCstMatch>(MI, MRI, OpcodeTriple, Val, K0, K1))
    return false;

  if (!K0->Value.isExactlyValue(0.0) || !K1->Value.isExactlyValue(1.0))
    return false;

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `AMDGPURegBankCombinerImpl::matchFPMinMaxToClamp`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`AMDGPURegBankCombinerImpl::matchFPMinMaxToClamp`。

### Lines 293-323: Conditional logic and checks
```cpp
  // For IEEE=false perform combine only when it's safe to assume that there are
  // no NaN inputs. Most often MI is marked with nnan fast math flag.
  // For IEEE=true consider NaN inputs. Only min(max(QNaN, 0.0), 1.0) evaluates
  // to 0.0 requires dx10_clamp = true.
  if ((getIEEE() && getDX10Clamp() && isFminnumIeee(MI) &&
       VT->isKnownNeverSNaN(Val)) ||
      VT->isKnownNeverNaN(MI.getOperand(0).getReg())) {
    Reg = Val;
    return true;
  }

  return false;
}

// Replacing fmed3(NaN, 0.0, 1.0) with clamp. Requires dx10_clamp = true.
// Val = SNaN only for ieee = true. It is important which operand is NaN.
// min(min(SNaN, 0.0), 1.0) = min(QNaN, 1.0) = 1.0
// min(min(SNaN, 1.0), 0.0) = min(QNaN, 0.0) = 0.0
// min(min(0.0, 1.0), SNaN) = min(0.0, SNaN) = QNaN
// Val = NaN,ieee = false or Val = QNaN,ieee = true
// min(min(NaN, 0.0), 1.0) = min(0.0, 1.0) = 0.0
// min(min(NaN, 1.0), 0.0) = min(1.0, 0.0) = 0.0
// min(min(0.0, 1.0), NaN) = min(0.0, NaN) = 0.0
bool AMDGPURegBankCombinerImpl::matchFPMed3ToClamp(MachineInstr &MI,
                                                   Register &Reg) const {
  // In llvm-ir, clamp is often represented as an intrinsic call to
  // @llvm.amdgcn.fmed3.f32(%Val, 0.0, 1.0). Check for other operand orders.
  MachineInstr *Src0 = getDefIgnoringCopies(MI.getOperand(1).getReg(), MRI);
  MachineInstr *Src1 = getDefIgnoringCopies(MI.getOperand(2).getReg(), MRI);
  MachineInstr *Src2 = getDefIgnoringCopies(MI.getOperand(3).getReg(), MRI);

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `AMDGPURegBankCombinerImpl::matchFPMed3ToClamp`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`AMDGPURegBankCombinerImpl::matchFPMed3ToClamp`。

### Lines 324-354: Declares getOperand
```cpp
  if (isFCst(Src0) && !isFCst(Src1))
    std::swap(Src0, Src1);
  if (isFCst(Src1) && !isFCst(Src2))
    std::swap(Src1, Src2);
  if (isFCst(Src0) && !isFCst(Src1))
    std::swap(Src0, Src1);
  if (!isClampZeroToOne(Src1, Src2))
    return false;

  Register Val = Src0->getOperand(0).getReg();

  auto isOp3Zero = [&]() {
    MachineInstr *Op3 = getDefIgnoringCopies(MI.getOperand(3).getReg(), MRI);
    if (Op3->getOpcode() == TargetOpcode::G_FCONSTANT)
      return Op3->getOperand(1).getFPImm()->isExactlyValue(0.0);
    return false;
  };
  // For IEEE=false perform combine only when it's safe to assume that there are
  // no NaN inputs. Most often MI is marked with nnan fast math flag.
  // For IEEE=true consider NaN inputs. Requires dx10_clamp = true. Safe to fold
  // when Val could be QNaN. If Val can also be SNaN third input should be 0.0.
  if (VT->isKnownNeverNaN(MI.getOperand(0).getReg()) ||
      (getIEEE() && getDX10Clamp() &&
       (VT->isKnownNeverSNaN(Val) || isOp3Zero()))) {
    Reg = Val;
    return true;
  }

  return false;
}

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend. Main symbols: `std::swap`.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。 主要符号：`std::swap`。

### Lines 355-386: Implements AMDGPURegBankCombinerImpl::applyClamp
```cpp
void AMDGPURegBankCombinerImpl::applyClamp(MachineInstr &MI,
                                           Register &Reg) const {
  B.buildInstr(AMDGPU::G_AMDGPU_CLAMP, {MI.getOperand(0)}, {Reg},
               MI.getFlags());
  MI.eraseFromParent();
}

void AMDGPURegBankCombinerImpl::applyMed3(MachineInstr &MI,
                                          Med3MatchInfo &MatchInfo) const {
  B.buildInstr(MatchInfo.Opc, {MI.getOperand(0)},
               {getAsVgpr(MatchInfo.Val0), getAsVgpr(MatchInfo.Val1),
                getAsVgpr(MatchInfo.Val2)},
               MI.getFlags());
  MI.eraseFromParent();
}

void AMDGPURegBankCombinerImpl::applyCanonicalizeZextShiftAmt(
    MachineInstr &MI, MachineInstr &Ext) const {
  unsigned ShOpc = MI.getOpcode();
  assert(ShOpc == AMDGPU::G_SHL || ShOpc == AMDGPU::G_LSHR ||
         ShOpc == AMDGPU::G_ASHR);
  assert(Ext.getOpcode() == AMDGPU::G_ZEXT);

  Register AmtReg = Ext.getOperand(1).getReg();
  Register ShDst = MI.getOperand(0).getReg();
  Register ShSrc = MI.getOperand(1).getReg();

  LLT ExtAmtTy = MRI.getType(Ext.getOperand(0).getReg());
  LLT AmtTy = MRI.getType(AmtReg);

  auto &RB = *MRI.getRegBank(AmtReg);

```
**EN:** This section contains concrete logic for AMDGPURegBankCombinerImpl::applyClamp. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPURegBankCombinerImpl::applyClamp`, `AMDGPURegBankCombinerImpl::applyMed3`, `AMDGPURegBankCombinerImpl::applyCanonicalizeZextShiftAmt`.
**CN:** 本节包含与 AMDGPURegBankCombinerImpl::applyClamp 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPURegBankCombinerImpl::applyClamp`, `AMDGPURegBankCombinerImpl::applyMed3`, `AMDGPURegBankCombinerImpl::applyCanonicalizeZextShiftAmt`。

### Lines 387-420: Declares buildAnyExt
```cpp
  auto NewExt = B.buildAnyExt(ExtAmtTy, AmtReg);
  auto Mask = B.buildConstant(
      ExtAmtTy, maskTrailingOnes<uint64_t>(AmtTy.getScalarSizeInBits()));
  auto And = B.buildAnd(ExtAmtTy, NewExt, Mask);
  B.buildInstr(ShOpc, {ShDst}, {ShSrc, And});

  MRI.setRegBank(NewExt.getReg(0), RB);
  MRI.setRegBank(Mask.getReg(0), RB);
  MRI.setRegBank(And.getReg(0), RB);
  MI.eraseFromParent();
}

bool AMDGPURegBankCombinerImpl::combineD16Load(MachineInstr &MI) const {
  Register Dst;
  MachineInstr *Load, *SextLoad;
  const int64_t CleanLo16 = 0xFFFFFFFFFFFF0000;
  const int64_t CleanHi16 = 0x000000000000FFFF;

  // Load lo
  if (mi_match(MI.getOperand(1).getReg(), MRI,
               m_GOr(m_GAnd(m_GBitcast(m_Reg(Dst)),
                            m_Copy(m_SpecificICst(CleanLo16))),
                     m_MInstr(Load)))) {

    if (Load->getOpcode() == AMDGPU::G_ZEXTLOAD) {
      const MachineMemOperand *MMO = *Load->memoperands_begin();
      unsigned LoadSize = MMO->getSizeInBits().getValue();
      if (LoadSize == 8)
        return applyD16Load(AMDGPU::G_AMDGPU_LOAD_D16_LO_U8, MI, Load, Dst);
      if (LoadSize == 16)
        return applyD16Load(AMDGPU::G_AMDGPU_LOAD_D16_LO, MI, Load, Dst);
      return false;
    }

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend. Main symbols: `AMDGPURegBankCombinerImpl::combineD16Load`.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。 主要符号：`AMDGPURegBankCombinerImpl::combineD16Load`。

### Lines 421-452: Conditional logic and checks
```cpp
    if (mi_match(
            Load, MRI,
            m_GAnd(m_MInstr(SextLoad), m_Copy(m_SpecificICst(CleanHi16))))) {
      if (SextLoad->getOpcode() != AMDGPU::G_SEXTLOAD)
        return false;

      const MachineMemOperand *MMO = *SextLoad->memoperands_begin();
      if (MMO->getSizeInBits().getValue() != 8)
        return false;

      return applyD16Load(AMDGPU::G_AMDGPU_LOAD_D16_LO_I8, MI, SextLoad, Dst);
    }

    return false;
  }

  // Load hi
  if (mi_match(MI.getOperand(1).getReg(), MRI,
               m_GOr(m_GAnd(m_GBitcast(m_Reg(Dst)),
                            m_Copy(m_SpecificICst(CleanHi16))),
                     m_GShl(m_MInstr(Load), m_Copy(m_SpecificICst(16)))))) {

    if (Load->getOpcode() == AMDGPU::G_ZEXTLOAD) {
      const MachineMemOperand *MMO = *Load->memoperands_begin();
      unsigned LoadSize = MMO->getSizeInBits().getValue();
      if (LoadSize == 8)
        return applyD16Load(AMDGPU::G_AMDGPU_LOAD_D16_HI_U8, MI, Load, Dst);
      if (LoadSize == 16)
        return applyD16Load(AMDGPU::G_AMDGPU_LOAD_D16_HI, MI, Load, Dst);
      return false;
    }

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 453-486: Conditional logic and checks
```cpp
    if (mi_match(
            Load, MRI,
            m_GAnd(m_MInstr(SextLoad), m_Copy(m_SpecificICst(CleanHi16))))) {
      if (SextLoad->getOpcode() != AMDGPU::G_SEXTLOAD)
        return false;
      const MachineMemOperand *MMO = *SextLoad->memoperands_begin();
      if (MMO->getSizeInBits().getValue() != 8)
        return false;

      return applyD16Load(AMDGPU::G_AMDGPU_LOAD_D16_HI_I8, MI, SextLoad, Dst);
    }

    return false;
  }

  return false;
}

bool AMDGPURegBankCombinerImpl::applyD16Load(
    unsigned D16Opc, MachineInstr &DstMI, MachineInstr *SmallLoad,
    Register SrcReg32ToOverwriteD16) const {
  B.buildInstr(D16Opc, {DstMI.getOperand(0).getReg()},
               {SmallLoad->getOperand(1).getReg(), SrcReg32ToOverwriteD16})
      .setMemRefs(SmallLoad->memoperands());
  DstMI.eraseFromParent();
  return true;
}

SIModeRegisterDefaults AMDGPURegBankCombinerImpl::getMode() const {
  return MF.getInfo<SIMachineFunctionInfo>()->getMode();
}

bool AMDGPURegBankCombinerImpl::getIEEE() const { return getMode().IEEE; }

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `AMDGPURegBankCombinerImpl::applyD16Load`, `AMDGPURegBankCombinerImpl::getMode`, `AMDGPURegBankCombinerImpl::getIEEE`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`AMDGPURegBankCombinerImpl::applyD16Load`, `AMDGPURegBankCombinerImpl::getMode`, `AMDGPURegBankCombinerImpl::getIEEE`。

### Lines 487-520: Declares class AMDGPURegBankCombiner
```cpp
bool AMDGPURegBankCombinerImpl::getDX10Clamp() const {
  return getMode().DX10Clamp;
}

bool AMDGPURegBankCombinerImpl::isFminnumIeee(const MachineInstr &MI) const {
  return MI.getOpcode() == AMDGPU::G_FMINNUM_IEEE;
}

bool AMDGPURegBankCombinerImpl::isFCst(MachineInstr *MI) const {
  return MI->getOpcode() == AMDGPU::G_FCONSTANT;
}

bool AMDGPURegBankCombinerImpl::isClampZeroToOne(MachineInstr *K0,
                                                 MachineInstr *K1) const {
  if (isFCst(K0) && isFCst(K1)) {
    const ConstantFP *KO_FPImm = K0->getOperand(1).getFPImm();
    const ConstantFP *K1_FPImm = K1->getOperand(1).getFPImm();
    return (KO_FPImm->isExactlyValue(0.0) && K1_FPImm->isExactlyValue(1.0)) ||
           (KO_FPImm->isExactlyValue(1.0) && K1_FPImm->isExactlyValue(0.0));
  }
  return false;
}

// Pass boilerplate
// ================

class AMDGPURegBankCombiner : public MachineFunctionPass {
public:
  static char ID;

  AMDGPURegBankCombiner(bool IsOptNone = false);

  StringRef getPassName() const override { return "AMDGPURegBankCombiner"; }

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `AMDGPURegBankCombiner`, `AMDGPURegBankCombinerImpl::getDX10Clamp`, `AMDGPURegBankCombinerImpl::isFminnumIeee`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`AMDGPURegBankCombiner`, `AMDGPURegBankCombinerImpl::getDX10Clamp`, `AMDGPURegBankCombinerImpl::isFminnumIeee`。

### Lines 521-548: Declares runOnMachineFunction
```cpp
  bool runOnMachineFunction(MachineFunction &MF) override;

  void getAnalysisUsage(AnalysisUsage &AU) const override;

private:
  bool IsOptNone;
  AMDGPURegBankCombinerImplRuleConfig RuleConfig;
};
} // end anonymous namespace

void AMDGPURegBankCombiner::getAnalysisUsage(AnalysisUsage &AU) const {
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

AMDGPURegBankCombiner::AMDGPURegBankCombiner(bool IsOptNone)
    : MachineFunctionPass(ID), IsOptNone(IsOptNone) {
  if (!RuleConfig.parseCommandLineOption())
    report_fatal_error("Invalid rule identifier");
}

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend. Main symbols: `AMDGPURegBankCombiner::getAnalysisUsage`, `MachineFunctionPass::getAnalysisUsage`, `AMDGPURegBankCombiner::AMDGPURegBankCombiner`.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。 主要符号：`AMDGPURegBankCombiner::getAnalysisUsage`, `MachineFunctionPass::getAnalysisUsage`, `AMDGPURegBankCombiner::AMDGPURegBankCombiner`。

### Lines 549-577: Implements AMDGPURegBankCombiner::runOnMachineFunction
```cpp
bool AMDGPURegBankCombiner::runOnMachineFunction(MachineFunction &MF) {
  if (MF.getProperties().hasFailedISel())
    return false;
  const Function &F = MF.getFunction();
  bool EnableOpt =
      MF.getTarget().getOptLevel() != CodeGenOptLevel::None && !skipFunction(F);

  const GCNSubtarget &ST = MF.getSubtarget<GCNSubtarget>();
  GISelValueTracking *VT =
      &getAnalysis<GISelValueTrackingAnalysisLegacy>().get(MF);

  const auto *LI = ST.getLegalizerInfo();
  MachineDominatorTree *MDT =
      IsOptNone ? nullptr
                : &getAnalysis<MachineDominatorTreeWrapperPass>().getDomTree();

  CombinerInfo CInfo(/*AllowIllegalOps*/ false, /*ShouldLegalizeIllegal*/ true,
                     LI, EnableOpt, F.hasOptSize(), F.hasMinSize());
  // Disable fixed-point iteration to reduce compile-time
  CInfo.MaxIterations = 1;
  CInfo.ObserverLvl = CombinerInfo::ObserverLevel::SinglePass;
  // RegBankSelect seems not to leave dead instructions, so a full DCE pass is
  // unnecessary.
  CInfo.EnableFullDCE = false;
  AMDGPURegBankCombinerImpl Impl(MF, CInfo, *VT, /*CSEInfo*/ nullptr,
                                 RuleConfig, ST, MDT, LI);
  return Impl.combineMachineInstrs();
}

```
**EN:** This section contains concrete logic for AMDGPURegBankCombiner::runOnMachineFunction. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPURegBankCombiner::runOnMachineFunction`.
**CN:** 本节包含与 AMDGPURegBankCombiner::runOnMachineFunction 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPURegBankCombiner::runOnMachineFunction`。

### Lines 578-589: Registers LLVM passes
```cpp
char AMDGPURegBankCombiner::ID = 0;
INITIALIZE_PASS_BEGIN(AMDGPURegBankCombiner, DEBUG_TYPE,
                      "Combine AMDGPU machine instrs after regbankselect",
                      false, false)
INITIALIZE_PASS_DEPENDENCY(GISelValueTrackingAnalysisLegacy)
INITIALIZE_PASS_END(AMDGPURegBankCombiner, DEBUG_TYPE,
                    "Combine AMDGPU machine instrs after regbankselect", false,
                    false)

FunctionPass *llvm::createAMDGPURegBankCombiner(bool IsOptNone) {
  return new AMDGPURegBankCombiner(IsOptNone);
}
```
**EN:** This section connects the implementation to LLVM pass registration so the AMDGPU-specific analysis or transformation can be constructed by the pass framework. Main symbols: `llvm::createAMDGPURegBankCombiner`.
**CN:** 本节把实现接入 LLVM 的 Pass 注册机制，使 AMDGPU 专用分析或变换能够被 Pass 框架创建和调用。 主要符号：`llvm::createAMDGPURegBankCombiner`。

## Key Concepts / 关键概念
- **Language / 语言**: C++ source
- **Primary symbols / 主要符号**: `AMDGPURegBankCombinerImpl`, `MinMaxMedOpc`, `Med3MatchInfo`, `AMDGPURegBankCombiner`, `AMDGPURegBankCombinerImpl::AMDGPURegBankCombinerImpl`, `AMDGPURegBankCombinerImpl::isVgprRegBank`
- **Main themes / 核心主题**: register management / 寄存器管理; instruction semantics / 指令语义; subtarget modeling / 子目标建模; legalization / 合法化
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"AMDGPU.h"`
- `"AMDGPULegalizerInfo.h"`
- `"AMDGPURegisterBankInfo.h"`
- `"GCNSubtarget.h"`
- `"MCTargetDesc/AMDGPUMCTargetDesc.h"`
- `"SIMachineFunctionInfo.h"`
- `"llvm/CodeGen/GlobalISel/Combiner.h"`
- `"llvm/CodeGen/GlobalISel/CombinerHelper.h"`
- `"llvm/CodeGen/GlobalISel/CombinerInfo.h"`
- `"llvm/CodeGen/GlobalISel/GIMatchTableExecutorImpl.h"`
- `"llvm/CodeGen/GlobalISel/GISelValueTracking.h"`
- `"llvm/CodeGen/GlobalISel/MIPatternMatch.h"`
- `"llvm/CodeGen/MachineDominators.h"`
- `"llvm/CodeGen/TargetPassConfig.h"`
- `"llvm/Target/TargetMachine.h"`
- `"AMDGPUGenPreLegalizeGICombiner.inc"`
- `"AMDGPUGenRegBankGICombiner.inc"`
