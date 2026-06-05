# AMDGPURegBankLegalize.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/AMDGPURegBankLegalize.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This source file implements AMDGPURegBankLegalize for the LLVM AMDGPU backend. It contains target-specific logic used during analysis, lowering, code generation, or pass execution. / 该源文件实现 LLVM AMDGPU 后端中的 AMDGPURegBankLegalize 相关功能。它包含分析、降低、代码生成或 Pass 执行过程中使用的目标专用逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-33: File banner, includes, and setup
```cpp
//===-- AMDGPURegBankLegalize.cpp -----------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
/// Lower G_ instructions that can't be inst-selected with register bank
/// assignment from AMDGPURegBankSelect based on machine uniformity info.
/// Given types on all operands, some register bank assignments require lowering
/// while others do not.
/// Note: cases where all register bank assignments would require lowering are
/// lowered in legalizer.
/// For example vgpr S64 G_AND requires lowering to S32 while sgpr S64 does not.
/// Eliminate sgpr S1 by lowering to sgpr S32.
//
//===----------------------------------------------------------------------===//

#include "AMDGPU.h"
#include "AMDGPUGlobalISelUtils.h"
#include "AMDGPURegBankLegalizeHelper.h"
#include "GCNSubtarget.h"
#include "llvm/CodeGen/GlobalISel/CSEInfo.h"
#include "llvm/CodeGen/GlobalISel/CSEMIRBuilder.h"
#include "llvm/CodeGen/GlobalISel/GenericMachineInstrs.h"
#include "llvm/CodeGen/GlobalISel/MIPatternMatch.h"
#include "llvm/CodeGen/GlobalISel/Utils.h"
#include "llvm/CodeGen/MachineFunctionPass.h"
#include "llvm/CodeGen/MachineUniformityAnalysis.h"
#include "llvm/CodeGen/TargetPassConfig.h"
#include "llvm/InitializePasses.h"

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。

### Lines 34-61: Preprocessor guards and macros
```cpp
#define DEBUG_TYPE "amdgpu-regbanklegalize"

using namespace llvm;
using namespace AMDGPU;
using namespace llvm::MIPatternMatch;

namespace {

// AMDGPU-specific pattern matchers
template <typename SrcTy>
inline UnaryOp_match<SrcTy, AMDGPU::G_AMDGPU_READANYLANE>
m_GAMDGPUReadAnyLane(const SrcTy &Src) {
  return UnaryOp_match<SrcTy, AMDGPU::G_AMDGPU_READANYLANE>(Src);
}

class AMDGPURegBankLegalize : public MachineFunctionPass {
public:
  static char ID;

public:
  AMDGPURegBankLegalize() : MachineFunctionPass(ID) {}

  bool runOnMachineFunction(MachineFunction &MF) override;

  StringRef getPassName() const override {
    return "AMDGPU Register Bank Legalize";
  }

```
**EN:** These lines define compile-time structure such as include guards, feature switches, or macros that shape how the file is compiled. Main symbols: `AMDGPURegBankLegalize`.
**CN:** 这些语句定义了编译期结构，例如 include guard、特性开关或宏，用于决定文件的编译方式。 主要符号：`AMDGPURegBankLegalize`。

### Lines 62-93: Registers LLVM passes
```cpp
  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.addRequired<TargetPassConfig>();
    AU.addRequired<GISelCSEAnalysisWrapperPass>();
    AU.addRequired<MachineUniformityAnalysisPass>();
    MachineFunctionPass::getAnalysisUsage(AU);
  }

  // If there were no phis and we do waterfall expansion machine verifier would
  // fail.
  MachineFunctionProperties getClearedProperties() const override {
    return MachineFunctionProperties().setNoPHIs();
  }
};

} // End anonymous namespace.

INITIALIZE_PASS_BEGIN(AMDGPURegBankLegalize, DEBUG_TYPE,
                      "AMDGPU Register Bank Legalize", false, false)
INITIALIZE_PASS_DEPENDENCY(TargetPassConfig)
INITIALIZE_PASS_DEPENDENCY(GISelCSEAnalysisWrapperPass)
INITIALIZE_PASS_DEPENDENCY(MachineUniformityAnalysisPass)
INITIALIZE_PASS_END(AMDGPURegBankLegalize, DEBUG_TYPE,
                    "AMDGPU Register Bank Legalize", false, false)

char AMDGPURegBankLegalize::ID = 0;

char &llvm::AMDGPURegBankLegalizeID = AMDGPURegBankLegalize::ID;

FunctionPass *llvm::createAMDGPURegBankLegalizePass() {
  return new AMDGPURegBankLegalize();
}

```
**EN:** This section connects the implementation to LLVM pass registration so the AMDGPU-specific analysis or transformation can be constructed by the pass framework. Main symbols: `MachineFunctionPass::getAnalysisUsage`, `llvm::createAMDGPURegBankLegalizePass`.
**CN:** 本节把实现接入 LLVM 的 Pass 注册机制，使 AMDGPU 专用分析或变换能够被 Pass 框架创建和调用。 主要符号：`MachineFunctionPass::getAnalysisUsage`, `llvm::createAMDGPURegBankLegalizePass`。

### Lines 94-120: Declares class AMDGPURegBankLegalizeCombiner
```cpp
const RegBankLegalizeRules &getRules(const GCNSubtarget &ST,
                                     MachineRegisterInfo &MRI) {
  static std::mutex GlobalMutex;
  static SmallDenseMap<unsigned, std::unique_ptr<RegBankLegalizeRules>>
      CacheForRuleSet;
  std::lock_guard<std::mutex> Lock(GlobalMutex);
  auto [It, Inserted] = CacheForRuleSet.try_emplace(ST.getGeneration());
  if (Inserted)
    It->second = std::make_unique<RegBankLegalizeRules>(ST, MRI);
  else
    It->second->refreshRefs(ST, MRI);
  return *It->second;
}

class AMDGPURegBankLegalizeCombiner {
  MachineIRBuilder &B;
  MachineRegisterInfo &MRI;
  const SIRegisterInfo &TRI;
  const RegisterBank *SgprRB;
  const RegisterBank *VgprRB;
  const RegisterBank *VccRB;

  static constexpr LLT S1 = LLT::scalar(1);
  static constexpr LLT S16 = LLT::scalar(16);
  static constexpr LLT S32 = LLT::scalar(32);
  static constexpr LLT S64 = LLT::scalar(64);

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `AMDGPURegBankLegalizeCombiner`, `LLT::scalar`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`AMDGPURegBankLegalizeCombiner`, `LLT::scalar`。

### Lines 121-149: Defines AMDGPURegBankLegalizeCombiner
```cpp
public:
  AMDGPURegBankLegalizeCombiner(MachineIRBuilder &B, const SIRegisterInfo &TRI,
                                const RegisterBankInfo &RBI)
      : B(B), MRI(*B.getMRI()), TRI(TRI),
        SgprRB(&RBI.getRegBank(AMDGPU::SGPRRegBankID)),
        VgprRB(&RBI.getRegBank(AMDGPU::VGPRRegBankID)),
        VccRB(&RBI.getRegBank(AMDGPU::VCCRegBankID)) {};

  bool isLaneMask(Register Reg);
  std::pair<MachineInstr *, Register> tryMatch(Register Src, unsigned Opcode);
  Register tryMatchUnmergeDefs(SmallVectorImpl<Register> &DefRegs);
  SmallVector<Register> tryMatchMergeReadAnyLane(GMergeLikeInstr *Merge);
  SmallVector<Register> getReadAnyLaneSrcs(Register Src);
  void replaceRegWithOrBuildCopy(Register Dst, Register Src);

  bool tryEliminateReadAnyLane(MachineInstr &Copy);
  void tryCombineCopy(MachineInstr &MI);
  void tryCombineS1AnyExt(MachineInstr &MI);
};

bool AMDGPURegBankLegalizeCombiner::isLaneMask(Register Reg) {
  const RegisterBank *RB = MRI.getRegBankOrNull(Reg);
  if (RB && RB->getID() == AMDGPU::VCCRegBankID)
    return true;

  const TargetRegisterClass *RC = MRI.getRegClassOrNull(Reg);
  return RC && TRI.isSGPRClass(RC) && MRI.getType(Reg) == LLT::scalar(1);
}

```
**EN:** This section contains concrete logic for AMDGPURegBankLegalizeCombiner. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPURegBankLegalizeCombiner::isLaneMask`, `LLT::scalar`.
**CN:** 本节包含与 AMDGPURegBankLegalizeCombiner 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPURegBankLegalizeCombiner::isLaneMask`, `LLT::scalar`。

### Lines 150-171: Implements AMDGPURegBankLegalizeCombiner::tryMatch
```cpp
std::pair<MachineInstr *, Register>
AMDGPURegBankLegalizeCombiner::tryMatch(Register Src, unsigned Opcode) {
  MachineInstr *MatchMI = MRI.getVRegDef(Src);
  if (MatchMI->getOpcode() != Opcode)
    return {nullptr, Register()};
  return {MatchMI, MatchMI->getOperand(1).getReg()};
}

// Check if all registers are from same unmerge and there is no shuffling.
// Returns the unmerge source if both conditions are met.
Register AMDGPURegBankLegalizeCombiner::tryMatchUnmergeDefs(
    SmallVectorImpl<Register> &DefRegs) {
  auto *UnMerge = getOpcodeDef<GUnmerge>(DefRegs[0], MRI);
  if (!UnMerge || UnMerge->getNumDefs() != DefRegs.size())
    return {};
  for (unsigned I = 1; I < DefRegs.size(); ++I) {
    if (UnMerge->getReg(I) != DefRegs[I])
      return {};
  }
  return UnMerge->getSourceReg();
}

```
**EN:** This section contains concrete logic for AMDGPURegBankLegalizeCombiner::tryMatch. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPURegBankLegalizeCombiner::tryMatch`, `AMDGPURegBankLegalizeCombiner::tryMatchUnmergeDefs`.
**CN:** 本节包含与 AMDGPURegBankLegalizeCombiner::tryMatch 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPURegBankLegalizeCombiner::tryMatch`, `AMDGPURegBankLegalizeCombiner::tryMatchUnmergeDefs`。

### Lines 172-201: Implements AMDGPURegBankLegalizeCombiner::tryMatchMergeReadAnyLane
```cpp
// Check if all merge sources are readanylanes and return the readanylane
// sources if they are.
SmallVector<Register> AMDGPURegBankLegalizeCombiner::tryMatchMergeReadAnyLane(
    GMergeLikeInstr *Merge) {
  SmallVector<Register> ReadAnyLaneSrcs;
  for (unsigned i = 0; i < Merge->getNumSources(); ++i) {
    Register Src;
    if (!mi_match(Merge->getSourceReg(i), MRI,
                  m_GAMDGPUReadAnyLane(m_Reg(Src))))
      return {};
    ReadAnyLaneSrcs.push_back(Src);
  }
  return ReadAnyLaneSrcs;
}

SmallVector<Register>
AMDGPURegBankLegalizeCombiner::getReadAnyLaneSrcs(Register Src) {
  // Src = G_AMDGPU_READANYLANE RALSrc
  Register RALSrc;
  if (mi_match(Src, MRI, m_GAMDGPUReadAnyLane(m_Reg(RALSrc))))
    return {RALSrc};

  // RALSrc = G_ANYEXT S16Src
  // TruncSrc = G_AMDGPU_READANYLANE RALSrc
  // Src = G_TRUNC TruncSrc
  if (mi_match(Src, MRI,
               m_GTrunc(m_GAMDGPUReadAnyLane(m_GAnyExt(m_Reg(RALSrc)))))) {
    return {RALSrc};
  }

```
**EN:** This section contains concrete logic for AMDGPURegBankLegalizeCombiner::tryMatchMergeReadAnyLane. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPURegBankLegalizeCombiner::tryMatchMergeReadAnyLane`, `AMDGPURegBankLegalizeCombiner::getReadAnyLaneSrcs`.
**CN:** 本节包含与 AMDGPURegBankLegalizeCombiner::tryMatchMergeReadAnyLane 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPURegBankLegalizeCombiner::tryMatchMergeReadAnyLane`, `AMDGPURegBankLegalizeCombiner::getReadAnyLaneSrcs`。

### Lines 202-234: Conditional logic and checks
```cpp
  // TruncSrc = G_AMDGPU_READANYLANE RALSrc
  // AextSrc = G_TRUNC TruncSrc
  // Src = G_ANYEXT AextSrc
  if (mi_match(Src, MRI,
               m_GAnyExt(m_GTrunc(m_GAMDGPUReadAnyLane(m_Reg(RALSrc)))))) {
    return {RALSrc};
  }

  // Sgpr0 = G_AMDGPU_READANYLANE Vgpr0
  // Sgpr1 = G_AMDGPU_READANYLANE Vgpr1
  // ...
  // Src = G_MERGE_LIKE Sgpr0, Sgpr1, ...
  // Dst = COPY Src
  if (auto *Merge = getOpcodeDef<GMergeLikeInstr>(Src, MRI)) {
    SmallVector<Register> ReadAnyLaneSrcs = tryMatchMergeReadAnyLane(Merge);
    if (ReadAnyLaneSrcs.empty())
      return {};

    // Vgpr0, Vgpr1, ... = G_UNMERGE_VALUES UnmergeSrc
    if (Register UnmergeSrc = tryMatchUnmergeDefs(ReadAnyLaneSrcs))
      return {UnmergeSrc};

    // Multiple ReadAnyLane vgpr sources, need to merge Vgpr0, Vgpr1, ...
    return ReadAnyLaneSrcs;
  }

  // SrcRegIdx = G_AMDGPU_READANYLANE RALElSrc
  // SourceReg G_MERGE_VALUES ..., SrcRegIdx, ...
  // ..., Src, ... = G_UNMERGE_VALUES SourceReg
  auto *UnMerge = getOpcodeDef<GUnmerge>(Src, MRI);
  if (!UnMerge)
    return {};

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 235-268: Declares findRegisterDefOperandIdx
```cpp
  int Idx = UnMerge->findRegisterDefOperandIdx(Src, nullptr);
  auto *Merge = getOpcodeDef<GMergeLikeInstr>(UnMerge->getSourceReg(), MRI);
  if (!Merge || UnMerge->getNumDefs() != Merge->getNumSources())
    return {};

  Register SrcRegIdx = Merge->getSourceReg(Idx);
  if (MRI.getType(Src) != MRI.getType(SrcRegIdx))
    return {};

  auto [RALEl, RALElSrc] = tryMatch(SrcRegIdx, AMDGPU::G_AMDGPU_READANYLANE);
  if (RALEl)
    return {RALElSrc};

  return {};
}

void AMDGPURegBankLegalizeCombiner::replaceRegWithOrBuildCopy(Register Dst,
                                                              Register Src) {
  if (Dst.isVirtual())
    MRI.replaceRegWith(Dst, Src);
  else
    B.buildCopy(Dst, Src);
}

bool AMDGPURegBankLegalizeCombiner::tryEliminateReadAnyLane(
    MachineInstr &Copy) {
  Register Dst = Copy.getOperand(0).getReg();
  Register Src = Copy.getOperand(1).getReg();

  // Skip non-vgpr Dst
  if (Dst.isVirtual() ? (MRI.getRegBankOrNull(Dst) != VgprRB)
                      : !TRI.isVGPR(MRI, Dst))
    return false;

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend. Main symbols: `AMDGPURegBankLegalizeCombiner::replaceRegWithOrBuildCopy`, `AMDGPURegBankLegalizeCombiner::tryEliminateReadAnyLane`.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。 主要符号：`AMDGPURegBankLegalizeCombiner::replaceRegWithOrBuildCopy`, `AMDGPURegBankLegalizeCombiner::tryEliminateReadAnyLane`。

### Lines 269-292: Declares getVRegDef
```cpp
  // Skip physical source registers and source registers with register class
  if (!Src.isVirtual() || MRI.getRegClassOrNull(Src))
    return false;

  Register RALDst = Src;
  MachineInstr &SrcMI = *MRI.getVRegDef(Src);
  if (SrcMI.getOpcode() == AMDGPU::G_BITCAST)
    RALDst = SrcMI.getOperand(1).getReg();

  B.setInstrAndDebugLoc(Copy);
  SmallVector<Register> ReadAnyLaneSrcRegs = getReadAnyLaneSrcs(RALDst);
  if (ReadAnyLaneSrcRegs.empty())
    return false;

  Register ReadAnyLaneSrc;
  if (ReadAnyLaneSrcRegs.size() == 1) {
    ReadAnyLaneSrc = ReadAnyLaneSrcRegs[0];
  } else {
    // Multiple readanylane sources without a common unmerge, merge them.
    auto Merge = B.buildMergeLikeInstr({VgprRB, MRI.getType(RALDst)},
                                       ReadAnyLaneSrcRegs);
    ReadAnyLaneSrc = Merge.getReg(0);
  }

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。

### Lines 293-323: Conditional logic and checks
```cpp
  if (SrcMI.getOpcode() != AMDGPU::G_BITCAST) {
    // Src = READANYLANE RALSrc     Src = READANYLANE RALSrc
    // Dst = Copy Src               $Dst = Copy Src
    // ->                           ->
    // Dst = RALSrc                 $Dst = Copy RALSrc
    replaceRegWithOrBuildCopy(Dst, ReadAnyLaneSrc);
  } else {
    // RALDst = READANYLANE RALSrc  RALDst = READANYLANE RALSrc
    // Src = G_BITCAST RALDst       Src = G_BITCAST RALDst
    // Dst = Copy Src               Dst = Copy Src
    // ->                          ->
    // NewVgpr = G_BITCAST RALDst   NewVgpr = G_BITCAST RALDst
    // Dst = NewVgpr                $Dst = Copy NewVgpr
    auto Bitcast = B.buildBitcast({VgprRB, MRI.getType(Src)}, ReadAnyLaneSrc);
    replaceRegWithOrBuildCopy(Dst, Bitcast.getReg(0));
  }

  eraseInstr(Copy, MRI);
  return true;
}

void AMDGPURegBankLegalizeCombiner::tryCombineCopy(MachineInstr &MI) {
  if (tryEliminateReadAnyLane(MI))
    return;

  Register Dst = MI.getOperand(0).getReg();
  Register Src = MI.getOperand(1).getReg();
  // Skip copies of physical registers.
  if (!Dst.isVirtual() || !Src.isVirtual())
    return;

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `AMDGPURegBankLegalizeCombiner::tryCombineCopy`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`AMDGPURegBankLegalizeCombiner::tryCombineCopy`。

### Lines 324-354: Conditional logic and checks
```cpp
  // This is a cross bank copy, sgpr S1 to lane mask.
  //
  // %Src:sgpr(s1) = G_TRUNC %TruncS32Src:sgpr(s32)
  // %Dst:lane-mask(s1) = COPY %Src:sgpr(s1)
  // ->
  // %BoolSrc:sgpr(s32) = G_AND %TruncS32Src:sgpr(s32), 1
  // %Dst:lane-mask(s1) = G_AMDGPU_COPY_VCC_SCC %BoolSrc:sgpr(s32)
  if (isLaneMask(Dst) && MRI.getRegBankOrNull(Src) == SgprRB) {
    auto [Trunc, TruncS32Src] = tryMatch(Src, AMDGPU::G_TRUNC);
    assert(Trunc && MRI.getType(TruncS32Src) == S32 &&
           "sgpr S1 must be result of G_TRUNC of sgpr S32");

    B.setInstr(MI);
    // Ensure that truncated bits in BoolSrc are 0.
    auto One = B.buildConstant({SgprRB, S32}, 1);
    auto BoolSrc = B.buildAnd({SgprRB, S32}, TruncS32Src, One);
    B.buildInstr(AMDGPU::G_AMDGPU_COPY_VCC_SCC, {Dst}, {BoolSrc});
    eraseInstr(MI, MRI);
  }
}

void AMDGPURegBankLegalizeCombiner::tryCombineS1AnyExt(MachineInstr &MI) {
  // %Src:sgpr(S1) = G_TRUNC %TruncSrc
  // %Dst = G_ANYEXT %Src:sgpr(S1)
  // ->
  // %Dst = G_... %TruncSrc
  Register Dst = MI.getOperand(0).getReg();
  Register Src = MI.getOperand(1).getReg();
  if (MRI.getType(Src) != S1)
    return;

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `AMDGPURegBankLegalizeCombiner::tryCombineS1AnyExt`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`AMDGPURegBankLegalizeCombiner::tryCombineS1AnyExt`。

### Lines 355-383: Declares tryMatch
```cpp
  auto [Trunc, TruncSrc] = tryMatch(Src, AMDGPU::G_TRUNC);
  if (!Trunc)
    return;

  LLT DstTy = MRI.getType(Dst);
  LLT TruncSrcTy = MRI.getType(TruncSrc);

  if (DstTy == TruncSrcTy) {
    MRI.replaceRegWith(Dst, TruncSrc);
    eraseInstr(MI, MRI);
    return;
  }

  B.setInstr(MI);

  if (DstTy == S32 && TruncSrcTy == S64) {
    auto Unmerge = B.buildUnmerge({SgprRB, S32}, TruncSrc);
    MRI.replaceRegWith(Dst, Unmerge.getReg(0));
    eraseInstr(MI, MRI);
    return;
  }

  if (DstTy == S64 && TruncSrcTy == S32) {
    B.buildMergeLikeInstr(MI.getOperand(0).getReg(),
                          {TruncSrc, B.buildUndef({SgprRB, S32})});
    eraseInstr(MI, MRI);
    return;
  }

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。

### Lines 384-417: Conditional logic and checks
```cpp
  if (DstTy == S32 && TruncSrcTy == S16) {
    B.buildAnyExt(Dst, TruncSrc);
    eraseInstr(MI, MRI);
    return;
  }

  if (DstTy == S16 && TruncSrcTy == S32) {
    B.buildTrunc(Dst, TruncSrc);
    eraseInstr(MI, MRI);
    return;
  }

  llvm_unreachable("missing anyext + trunc combine");
}

// Search through MRI for virtual registers with sgpr register bank and S1 LLT.
[[maybe_unused]] static Register getAnySgprS1(const MachineRegisterInfo &MRI) {
  const LLT S1 = LLT::scalar(1);
  for (unsigned i = 0; i < MRI.getNumVirtRegs(); ++i) {
    Register Reg = Register::index2VirtReg(i);
    if (MRI.def_empty(Reg) || MRI.getType(Reg) != S1)
      continue;

    const RegisterBank *RB = MRI.getRegBankOrNull(Reg);
    if (RB && RB->getID() == AMDGPU::SGPRRegBankID) {
      LLVM_DEBUG(dbgs() << "Warning: detected sgpr S1 register in: ";
                 MRI.getVRegDef(Reg)->dump(););
      return Reg;
    }
  }

  return {};
}

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `LLT::scalar`, `Register::index2VirtReg`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`LLT::scalar`, `Register::index2VirtReg`。

### Lines 418-450: Implements AMDGPURegBankLegalize::runOnMachineFunction
```cpp
bool AMDGPURegBankLegalize::runOnMachineFunction(MachineFunction &MF) {
  if (MF.getProperties().hasFailedISel())
    return false;

  // Setup the instruction builder with CSE.
  const TargetPassConfig &TPC = getAnalysis<TargetPassConfig>();
  GISelCSEAnalysisWrapper &Wrapper =
      getAnalysis<GISelCSEAnalysisWrapperPass>().getCSEWrapper();
  GISelCSEInfo &CSEInfo = Wrapper.get(TPC.getCSEConfig());
  GISelObserverWrapper Observer;
  Observer.addObserver(&CSEInfo);

  CSEMIRBuilder B(MF);
  B.setCSEInfo(&CSEInfo);
  B.setChangeObserver(Observer);

  RAIIDelegateInstaller DelegateInstaller(MF, &Observer);
  RAIIMFObserverInstaller MFObserverInstaller(MF, Observer);

  const GCNSubtarget &ST = MF.getSubtarget<GCNSubtarget>();
  MachineRegisterInfo &MRI = MF.getRegInfo();
  const RegisterBankInfo &RBI = *ST.getRegBankInfo();
  const MachineUniformityInfo &MUI =
      getAnalysis<MachineUniformityAnalysisPass>().getUniformityInfo();

  // RegBankLegalizeRules is initialized with assigning sets of IDs to opcodes.
  const RegBankLegalizeRules &RBLRules = getRules(ST, MRI);

  // Logic that does legalization based on IDs assigned to Opcode.
  RegBankLegalizeHelper RBLHelper(B, MUI, RBI, RBLRules);

  SmallVector<MachineInstr *> AllInst;

```
**EN:** This section contains concrete logic for AMDGPURegBankLegalize::runOnMachineFunction. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPURegBankLegalize::runOnMachineFunction`.
**CN:** 本节包含与 AMDGPURegBankLegalize::runOnMachineFunction 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPURegBankLegalize::runOnMachineFunction`。

### Lines 451-481: Conditional logic and checks
```cpp
  for (MachineBasicBlock &MBB : MF) {
    for (MachineInstr &MI : MBB) {
      AllInst.push_back(&MI);
    }
  }

  for (MachineInstr *MI : AllInst) {
    if (!MI->isPreISelOpcode())
      continue;

    if (!RBLHelper.findRuleAndApplyMapping(*MI))
      return false;
  }

  // Sgpr S1 clean up combines:
  // - Sgpr S1(S32) to sgpr S1(S32) Copy: anyext + trunc combine.
  //   In RegBankLegalize 'S1 Dst' are legalized into S32 as
  //   'S1Dst = Trunc S32Dst' and 'S1 Src' into 'S32Src = Anyext S1Src'.
  //   S1 Truncs and Anyexts that come from legalizer, that can have non-S32
  //   types e.g. S16 = Anyext S1 or S1 = Trunc S64, will also be cleaned up.
  // - Sgpr S1(S32) to vcc Copy: G_AMDGPU_COPY_VCC_SCC combine.
  //   Divergent instruction uses sgpr S1 as input that should be lane mask(vcc)
  //   Legalizing this use creates sgpr S1(S32) to vcc Copy.

  // Note: Remaining S1 copies, S1s are either sgpr S1(S32) or vcc S1:
  // - Vcc to vcc Copy: nothing to do here, just a regular copy.
  // - Vcc to sgpr S1 Copy: Should not exist in a form of COPY instruction(*).
  //   Note: For 'uniform-in-vcc to sgpr-S1 copy' G_AMDGPU_COPY_SCC_VCC is used
  //   instead. When only available instruction creates vcc result, use of
  //   UniformInVcc results in creating G_AMDGPU_COPY_SCC_VCC.

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 482-508: Declares Combiner
```cpp
  // (*)Explanation for 'sgpr S1(uniform) = COPY vcc(divergent)':
  // Copy from divergent to uniform register indicates an error in either:
  // - Uniformity analysis: Uniform instruction has divergent input. If one of
  //   the inputs is divergent, instruction should be divergent!
  // - RegBankLegalizer not executing in waterfall loop (missing implementation)

  AMDGPURegBankLegalizeCombiner Combiner(B, *ST.getRegisterInfo(), RBI);

  for (MachineBasicBlock &MBB : MF) {
    for (MachineInstr &MI : make_early_inc_range(MBB)) {
      if (MI.getOpcode() == AMDGPU::COPY) {
        Combiner.tryCombineCopy(MI);
        continue;
      }
      if (MI.getOpcode() == AMDGPU::G_ANYEXT) {
        Combiner.tryCombineS1AnyExt(MI);
        continue;
      }
    }
  }

  assert(!getAnySgprS1(MRI).isValid() &&
         "Registers with sgpr reg bank and S1 LLT are not legal after "
         "AMDGPURegBankLegalize. Should lower to sgpr S32");

  return true;
}
```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。

## Key Concepts / 关键概念
- **Language / 语言**: C++ source
- **Primary symbols / 主要符号**: `AMDGPURegBankLegalize`, `AMDGPURegBankLegalizeCombiner`, `MachineFunctionPass::getAnalysisUsage`, `llvm::createAMDGPURegBankLegalizePass`, `LLT::scalar`, `AMDGPURegBankLegalizeCombiner::isLaneMask`
- **Main themes / 核心主题**: register management / 寄存器管理; instruction semantics / 指令语义; subtarget modeling / 子目标建模; lowering / 降低
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"AMDGPU.h"`
- `"AMDGPUGlobalISelUtils.h"`
- `"AMDGPURegBankLegalizeHelper.h"`
- `"GCNSubtarget.h"`
- `"llvm/CodeGen/GlobalISel/CSEInfo.h"`
- `"llvm/CodeGen/GlobalISel/CSEMIRBuilder.h"`
- `"llvm/CodeGen/GlobalISel/GenericMachineInstrs.h"`
- `"llvm/CodeGen/GlobalISel/MIPatternMatch.h"`
- `"llvm/CodeGen/GlobalISel/Utils.h"`
- `"llvm/CodeGen/MachineFunctionPass.h"`
- `"llvm/CodeGen/MachineUniformityAnalysis.h"`
- `"llvm/CodeGen/TargetPassConfig.h"`
- `"llvm/InitializePasses.h"`
