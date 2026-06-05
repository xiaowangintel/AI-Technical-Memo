# AMDGPUPreLegalizerCombiner.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/AMDGPUPreLegalizerCombiner.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This source file implements AMDGPUPreLegalizerCombiner for the LLVM AMDGPU backend. It contains target-specific logic used during analysis, lowering, code generation, or pass execution. / 该源文件实现 LLVM AMDGPU 后端中的 AMDGPUPreLegalizerCombiner 相关功能。它包含分析、降低、代码生成或 Pass 执行过程中使用的目标专用逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-33: File banner, includes, and setup
```cpp
//=== lib/CodeGen/GlobalISel/AMDGPUPreLegalizerCombiner.cpp ---------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This pass does combining of machine instructions at the generic MI level,
// before the legalizer.
//
//===----------------------------------------------------------------------===//

#include "AMDGPU.h"
#include "AMDGPUCombinerHelper.h"
#include "AMDGPULegalizerInfo.h"
#include "GCNSubtarget.h"
#include "MCTargetDesc/AMDGPUMCTargetDesc.h"
#include "llvm/CodeGen/GlobalISel/CSEInfo.h"
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

### Lines 34-62: Header dependencies and setup
```cpp
#define DEBUG_TYPE "amdgpu-prelegalizer-combiner"

using namespace llvm;
using namespace MIPatternMatch;
namespace {

#define GET_GICOMBINER_TYPES
#include "AMDGPUGenPreLegalizeGICombiner.inc"
#undef GET_GICOMBINER_TYPES

class AMDGPUPreLegalizerCombinerImpl : public Combiner {
protected:
  const AMDGPUPreLegalizerCombinerImplRuleConfig &RuleConfig;
  const GCNSubtarget &STI;
  const AMDGPUCombinerHelper Helper;

public:
  AMDGPUPreLegalizerCombinerImpl(
      MachineFunction &MF, CombinerInfo &CInfo, GISelValueTracking &VT,
      GISelCSEInfo *CSEInfo,
      const AMDGPUPreLegalizerCombinerImplRuleConfig &RuleConfig,
      const GCNSubtarget &STI, MachineDominatorTree *MDT,
      const LegalizerInfo *LI);

  static const char *getName() { return "AMDGPUPreLegalizerCombinerImpl"; }

  bool tryCombineAllImpl(MachineInstr &MI) const;
  bool tryCombineAll(MachineInstr &I) const override;

```
**EN:** These includes pull in the LLVM and AMDGPU declarations required by the rest of the file, making later symbols and helper APIs available. Main symbols: `AMDGPUPreLegalizerCombinerImpl`.
**CN:** 这些头文件引入本文件后续实现所需的 LLVM 与 AMDGPU 声明，使后面的符号和辅助 API 可以直接使用。 主要符号：`AMDGPUPreLegalizerCombinerImpl`。

### Lines 63-89: Header dependencies and setup
```cpp
  struct ClampI64ToI16MatchInfo {
    int64_t Cmp1 = 0;
    int64_t Cmp2 = 0;
    Register Origin;
  };

  bool matchClampI64ToI16(MachineInstr &MI, const MachineRegisterInfo &MRI,
                          const MachineFunction &MF,
                          ClampI64ToI16MatchInfo &MatchInfo) const;

  void applyClampI64ToI16(MachineInstr &MI,
                          const ClampI64ToI16MatchInfo &MatchInfo) const;

private:
#define GET_GICOMBINER_CLASS_MEMBERS
#define AMDGPUSubtarget GCNSubtarget
#include "AMDGPUGenPreLegalizeGICombiner.inc"
#undef GET_GICOMBINER_CLASS_MEMBERS
#undef AMDGPUSubtarget
};

#define GET_GICOMBINER_IMPL
#define AMDGPUSubtarget GCNSubtarget
#include "AMDGPUGenPreLegalizeGICombiner.inc"
#undef AMDGPUSubtarget
#undef GET_GICOMBINER_IMPL

```
**EN:** These includes pull in the LLVM and AMDGPU declarations required by the rest of the file, making later symbols and helper APIs available. Main symbols: `ClampI64ToI16MatchInfo`.
**CN:** 这些头文件引入本文件后续实现所需的 LLVM 与 AMDGPU 声明，使后面的符号和辅助 API 可以直接使用。 主要符号：`ClampI64ToI16MatchInfo`。

### Lines 90-119: Header dependencies and setup
```cpp
AMDGPUPreLegalizerCombinerImpl::AMDGPUPreLegalizerCombinerImpl(
    MachineFunction &MF, CombinerInfo &CInfo, GISelValueTracking &VT,
    GISelCSEInfo *CSEInfo,
    const AMDGPUPreLegalizerCombinerImplRuleConfig &RuleConfig,
    const GCNSubtarget &STI, MachineDominatorTree *MDT, const LegalizerInfo *LI)
    : Combiner(MF, CInfo, &VT, CSEInfo), RuleConfig(RuleConfig), STI(STI),
      Helper(Observer, B, /*IsPreLegalize*/ true, &VT, MDT, LI, STI),
#define GET_GICOMBINER_CONSTRUCTOR_INITS
#include "AMDGPUGenPreLegalizeGICombiner.inc"
#undef GET_GICOMBINER_CONSTRUCTOR_INITS
{
}

bool AMDGPUPreLegalizerCombinerImpl::tryCombineAll(MachineInstr &MI) const {
  if (tryCombineAllImpl(MI))
    return true;

  switch (MI.getOpcode()) {
  case TargetOpcode::G_SHUFFLE_VECTOR:
    return Helper.tryCombineShuffleVector(MI);
  }

  return false;
}

bool AMDGPUPreLegalizerCombinerImpl::matchClampI64ToI16(
    MachineInstr &MI, const MachineRegisterInfo &MRI, const MachineFunction &MF,
    ClampI64ToI16MatchInfo &MatchInfo) const {
  assert(MI.getOpcode() == TargetOpcode::G_TRUNC && "Invalid instruction!");

```
**EN:** These includes pull in the LLVM and AMDGPU declarations required by the rest of the file, making later symbols and helper APIs available. Main symbols: `AMDGPUPreLegalizerCombinerImpl::AMDGPUPreLegalizerCombinerImpl`, `AMDGPUPreLegalizerCombinerImpl::tryCombineAll`, `AMDGPUPreLegalizerCombinerImpl::matchClampI64ToI16`.
**CN:** 这些头文件引入本文件后续实现所需的 LLVM 与 AMDGPU 声明，使后面的符号和辅助 API 可以直接使用。 主要符号：`AMDGPUPreLegalizerCombinerImpl::AMDGPUPreLegalizerCombinerImpl`, `AMDGPUPreLegalizerCombinerImpl::tryCombineAll`, `AMDGPUPreLegalizerCombinerImpl::matchClampI64ToI16`。

### Lines 120-148: Declares getType
```cpp
  // Try to find a pattern where an i64 value should get clamped to short.
  const LLT SrcType = MRI.getType(MI.getOperand(1).getReg());
  if (SrcType != LLT::scalar(64))
    return false;

  const LLT DstType = MRI.getType(MI.getOperand(0).getReg());
  if (DstType != LLT::scalar(16))
    return false;

  Register Base;

  auto IsApplicableForCombine = [&MatchInfo]() -> bool {
    const auto Cmp1 = MatchInfo.Cmp1;
    const auto Cmp2 = MatchInfo.Cmp2;
    const auto Diff = std::abs(Cmp2 - Cmp1);

    // If the difference between both comparison values is 0 or 1, there is no
    // need to clamp.
    if (Diff == 0 || Diff == 1)
      return false;

    const int64_t Min = std::numeric_limits<int16_t>::min();
    const int64_t Max = std::numeric_limits<int16_t>::max();

    // Check if the comparison values are between SHORT_MIN and SHORT_MAX.
    return ((Cmp2 >= Cmp1 && Cmp1 >= Min && Cmp2 <= Max) ||
            (Cmp1 >= Cmp2 && Cmp1 <= Max && Cmp2 >= Min));
  };

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend. Main symbols: `LLT::scalar`, `std::abs`.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。 主要符号：`LLT::scalar`, `std::abs`。

### Lines 149-182: Conditional logic and checks
```cpp
  // Try to match a combination of min / max MIR opcodes.
  if (mi_match(MI.getOperand(1).getReg(), MRI,
               m_GSMin(m_Reg(Base), m_ICst(MatchInfo.Cmp1)))) {
    if (mi_match(Base, MRI,
                 m_GSMax(m_Reg(MatchInfo.Origin), m_ICst(MatchInfo.Cmp2)))) {
      return IsApplicableForCombine();
    }
  }

  if (mi_match(MI.getOperand(1).getReg(), MRI,
               m_GSMax(m_Reg(Base), m_ICst(MatchInfo.Cmp1)))) {
    if (mi_match(Base, MRI,
                 m_GSMin(m_Reg(MatchInfo.Origin), m_ICst(MatchInfo.Cmp2)))) {
      return IsApplicableForCombine();
    }
  }

  return false;
}

// We want to find a combination of instructions that
// gets generated when an i64 gets clamped to i16.
// The corresponding pattern is:
// G_MAX / G_MAX for i16 <= G_TRUNC i64.
// This can be efficiently written as following:
// v_cvt_pk_i16_i32 v0, v0, v1
// v_med3_i32 v0, Clamp_Min, v0, Clamp_Max
void AMDGPUPreLegalizerCombinerImpl::applyClampI64ToI16(
    MachineInstr &MI, const ClampI64ToI16MatchInfo &MatchInfo) const {

  Register Src = MatchInfo.Origin;
  assert(MI.getMF()->getRegInfo().getType(Src) == LLT::scalar(64));
  const LLT S32 = LLT::scalar(32);

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `AMDGPUPreLegalizerCombinerImpl::applyClampI64ToI16`, `LLT::scalar`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`AMDGPUPreLegalizerCombinerImpl::applyClampI64ToI16`, `LLT::scalar`。

### Lines 183-215: Declares class AMDGPUPreLegalizerCombiner
```cpp
  auto Unmerge = B.buildUnmerge(S32, Src);

  assert(MI.getOpcode() != AMDGPU::G_AMDGPU_CVT_PK_I16_I32);

  const LLT V2S16 = LLT::fixed_vector(2, 16);
  auto CvtPk =
      B.buildInstr(AMDGPU::G_AMDGPU_CVT_PK_I16_I32, {V2S16},
                   {Unmerge.getReg(0), Unmerge.getReg(1)}, MI.getFlags());

  auto MinBoundary = std::min(MatchInfo.Cmp1, MatchInfo.Cmp2);
  auto MaxBoundary = std::max(MatchInfo.Cmp1, MatchInfo.Cmp2);
  auto MinBoundaryDst = B.buildConstant(S32, MinBoundary);
  auto MaxBoundaryDst = B.buildConstant(S32, MaxBoundary);

  auto Bitcast = B.buildBitcast({S32}, CvtPk);

  auto Med3 = B.buildInstr(
      AMDGPU::G_AMDGPU_SMED3, {S32},
      {MinBoundaryDst.getReg(0), Bitcast.getReg(0), MaxBoundaryDst.getReg(0)},
      MI.getFlags());

  B.buildTrunc(MI.getOperand(0).getReg(), Med3);

  MI.eraseFromParent();
}

// Pass boilerplate
// ================

class AMDGPUPreLegalizerCombiner : public MachineFunctionPass {
public:
  static char ID;

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `AMDGPUPreLegalizerCombiner`, `LLT::fixed_vector`, `std::min`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`AMDGPUPreLegalizerCombiner`, `LLT::fixed_vector`, `std::min`。

### Lines 216-247: Declares AMDGPUPreLegalizerCombiner
```cpp
  AMDGPUPreLegalizerCombiner(bool IsOptNone = false);

  StringRef getPassName() const override {
    return "AMDGPUPreLegalizerCombiner";
  }

  bool runOnMachineFunction(MachineFunction &MF) override;

  void getAnalysisUsage(AnalysisUsage &AU) const override;

private:
  bool IsOptNone;
  AMDGPUPreLegalizerCombinerImplRuleConfig RuleConfig;
};
} // end anonymous namespace

void AMDGPUPreLegalizerCombiner::getAnalysisUsage(AnalysisUsage &AU) const {
  AU.addRequired<TargetPassConfig>();
  AU.setPreservesCFG();
  getSelectionDAGFallbackAnalysisUsage(AU);
  AU.addRequired<GISelValueTrackingAnalysisLegacy>();
  AU.addPreserved<GISelValueTrackingAnalysisLegacy>();
  if (!IsOptNone) {
    AU.addRequired<MachineDominatorTreeWrapperPass>();
    AU.addPreserved<MachineDominatorTreeWrapperPass>();
  }

  AU.addRequired<GISelCSEAnalysisWrapperPass>();
  AU.addPreserved<GISelCSEAnalysisWrapperPass>();
  MachineFunctionPass::getAnalysisUsage(AU);
}

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend. Main symbols: `AMDGPUPreLegalizerCombiner::getAnalysisUsage`, `MachineFunctionPass::getAnalysisUsage`.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。 主要符号：`AMDGPUPreLegalizerCombiner::getAnalysisUsage`, `MachineFunctionPass::getAnalysisUsage`。

### Lines 248-268: Implements AMDGPUPreLegalizerCombiner::AMDGPUPreLegalizerCombiner
```cpp
AMDGPUPreLegalizerCombiner::AMDGPUPreLegalizerCombiner(bool IsOptNone)
    : MachineFunctionPass(ID), IsOptNone(IsOptNone) {
  if (!RuleConfig.parseCommandLineOption())
    report_fatal_error("Invalid rule identifier");
}

bool AMDGPUPreLegalizerCombiner::runOnMachineFunction(MachineFunction &MF) {
  if (MF.getProperties().hasFailedISel())
    return false;
  auto *TPC = &getAnalysis<TargetPassConfig>();
  const Function &F = MF.getFunction();
  bool EnableOpt =
      MF.getTarget().getOptLevel() != CodeGenOptLevel::None && !skipFunction(F);
  GISelValueTracking *VT =
      &getAnalysis<GISelValueTrackingAnalysisLegacy>().get(MF);

  // Enable CSE.
  GISelCSEAnalysisWrapper &Wrapper =
      getAnalysis<GISelCSEAnalysisWrapperPass>().getCSEWrapper();
  auto *CSEInfo = &Wrapper.get(TPC->getCSEConfig());

```
**EN:** This section contains concrete logic for AMDGPUPreLegalizerCombiner::AMDGPUPreLegalizerCombiner. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPUPreLegalizerCombiner::AMDGPUPreLegalizerCombiner`, `AMDGPUPreLegalizerCombiner::runOnMachineFunction`.
**CN:** 本节包含与 AMDGPUPreLegalizerCombiner::AMDGPUPreLegalizerCombiner 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPUPreLegalizerCombiner::AMDGPUPreLegalizerCombiner`, `AMDGPUPreLegalizerCombiner::runOnMachineFunction`。

### Lines 269-298: Registers LLVM passes
```cpp
  const GCNSubtarget &STI = MF.getSubtarget<GCNSubtarget>();
  MachineDominatorTree *MDT =
      IsOptNone ? nullptr
                : &getAnalysis<MachineDominatorTreeWrapperPass>().getDomTree();
  CombinerInfo CInfo(/*AllowIllegalOps*/ true, /*ShouldLegalizeIllegal*/ false,
                     nullptr, EnableOpt, F.hasOptSize(), F.hasMinSize());
  // Disable fixed-point iteration to reduce compile-time
  CInfo.MaxIterations = 1;
  CInfo.ObserverLvl = CombinerInfo::ObserverLevel::SinglePass;
  // This is the first Combiner, so the input IR might contain dead
  // instructions.
  CInfo.EnableFullDCE = true;
  AMDGPUPreLegalizerCombinerImpl Impl(MF, CInfo, *VT, CSEInfo, RuleConfig, STI,
                                      MDT, STI.getLegalizerInfo());
  return Impl.combineMachineInstrs();
}

char AMDGPUPreLegalizerCombiner::ID = 0;
INITIALIZE_PASS_BEGIN(AMDGPUPreLegalizerCombiner, DEBUG_TYPE,
                      "Combine AMDGPU machine instrs before legalization",
                      false, false)
INITIALIZE_PASS_DEPENDENCY(TargetPassConfig)
INITIALIZE_PASS_DEPENDENCY(GISelValueTrackingAnalysisLegacy)
INITIALIZE_PASS_END(AMDGPUPreLegalizerCombiner, DEBUG_TYPE,
                    "Combine AMDGPU machine instrs before legalization", false,
                    false)

FunctionPass *llvm::createAMDGPUPreLegalizeCombiner(bool IsOptNone) {
  return new AMDGPUPreLegalizerCombiner(IsOptNone);
}
```
**EN:** This section connects the implementation to LLVM pass registration so the AMDGPU-specific analysis or transformation can be constructed by the pass framework. Main symbols: `llvm::createAMDGPUPreLegalizeCombiner`.
**CN:** 本节把实现接入 LLVM 的 Pass 注册机制，使 AMDGPU 专用分析或变换能够被 Pass 框架创建和调用。 主要符号：`llvm::createAMDGPUPreLegalizeCombiner`。

## Key Concepts / 关键概念
- **Language / 语言**: C++ source
- **Primary symbols / 主要符号**: `AMDGPUPreLegalizerCombinerImpl`, `ClampI64ToI16MatchInfo`, `AMDGPUPreLegalizerCombiner`, `AMDGPUPreLegalizerCombinerImpl::AMDGPUPreLegalizerCombinerImpl`, `AMDGPUPreLegalizerCombinerImpl::tryCombineAll`, `AMDGPUPreLegalizerCombinerImpl::matchClampI64ToI16`
- **Main themes / 核心主题**: register management / 寄存器管理; instruction semantics / 指令语义; subtarget modeling / 子目标建模; legalization / 合法化
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"AMDGPU.h"`
- `"AMDGPUCombinerHelper.h"`
- `"AMDGPULegalizerInfo.h"`
- `"GCNSubtarget.h"`
- `"MCTargetDesc/AMDGPUMCTargetDesc.h"`
- `"llvm/CodeGen/GlobalISel/CSEInfo.h"`
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
