# AMDGPUGlobalISelUtils.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/AMDGPUGlobalISelUtils.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This source file implements AMDGPUGlobalISelUtils for the LLVM backend utilities. It contains target-specific logic used during analysis, lowering, code generation, or pass execution. / 该源文件实现 LLVM 后端工具中的 AMDGPUGlobalISelUtils 相关功能。它包含分析、降低、代码生成或 Pass 执行过程中使用的目标专用逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-24: File banner, includes, and setup
```cpp
//===- AMDGPUGlobalISelUtils.cpp ---------------------------------*- C++ -*-==//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "AMDGPUGlobalISelUtils.h"
#include "AMDGPURegisterBankInfo.h"
#include "MCTargetDesc/AMDGPUMCTargetDesc.h"
#include "llvm/ADT/DenseSet.h"
#include "llvm/CodeGen/GlobalISel/GISelValueTracking.h"
#include "llvm/CodeGen/GlobalISel/GenericMachineInstrs.h"
#include "llvm/CodeGen/GlobalISel/MIPatternMatch.h"
#include "llvm/CodeGen/GlobalISel/MachineIRBuilder.h"
#include "llvm/CodeGenTypes/LowLevelType.h"
#include "llvm/IR/Constants.h"
#include "llvm/IR/IntrinsicsAMDGPU.h"

using namespace llvm;
using namespace AMDGPU;
using namespace MIPatternMatch;

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。

### Lines 25-40: Implements AMDGPU::getBaseWithConstantOffset
```cpp
std::pair<Register, unsigned>
AMDGPU::getBaseWithConstantOffset(MachineRegisterInfo &MRI, Register Reg,
                                  GISelValueTracking *ValueTracking,
                                  bool CheckNUW) {
  MachineInstr *Def = getDefIgnoringCopies(Reg, MRI);
  if (Def->getOpcode() == TargetOpcode::G_CONSTANT) {
    unsigned Offset;
    const MachineOperand &Op = Def->getOperand(1);
    if (Op.isImm())
      Offset = Op.getImm();
    else
      Offset = Op.getCImm()->getZExtValue();

    return std::pair(Register(), Offset);
  }

```
**EN:** This section contains concrete logic for AMDGPU::getBaseWithConstantOffset. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPU::getBaseWithConstantOffset`, `std::pair`.
**CN:** 本节包含与 AMDGPU::getBaseWithConstantOffset 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPU::getBaseWithConstantOffset`, `std::pair`。

### Lines 41-63: Conditional logic and checks
```cpp
  int64_t Offset;
  if (Def->getOpcode() == TargetOpcode::G_ADD) {
    // A 32-bit (address + offset) should not cause unsigned 32-bit integer
    // wraparound, because s_load instructions perform the addition in 64 bits.
    if (CheckNUW && !Def->getFlag(MachineInstr::NoUWrap)) {
      assert(MRI.getType(Reg).getScalarSizeInBits() == 32);
      return std::pair(Reg, 0);
    }
    // TODO: Handle G_OR used for add case
    if (mi_match(Def->getOperand(2).getReg(), MRI, m_ICst(Offset)))
      return std::pair(Def->getOperand(1).getReg(), Offset);

    // FIXME: matcher should ignore copies
    if (mi_match(Def->getOperand(2).getReg(), MRI, m_Copy(m_ICst(Offset))))
      return std::pair(Def->getOperand(1).getReg(), Offset);
  }

  Register Base;
  if (ValueTracking && mi_match(Reg, MRI, m_GOr(m_Reg(Base), m_ICst(Offset))) &&
      ValueTracking->maskedValueIsZero(Base,
                                       APInt(32, Offset, /*isSigned=*/true)))
    return std::pair(Base, Offset);

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `std::pair`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`std::pair`。

### Lines 64-85: Conditional logic and checks
```cpp
  // Handle G_PTRTOINT (G_PTR_ADD base, const) case
  if (Def->getOpcode() == TargetOpcode::G_PTRTOINT) {
    MachineInstr *Base;
    if (mi_match(Def->getOperand(1).getReg(), MRI,
                 m_GPtrAdd(m_MInstr(Base), m_ICst(Offset)))) {
      // If Base was int converted to pointer, simply return int and offset.
      if (Base->getOpcode() == TargetOpcode::G_INTTOPTR)
        return std::pair(Base->getOperand(1).getReg(), Offset);

      // Register returned here will be of pointer type.
      return std::pair(Base->getOperand(0).getReg(), Offset);
    }
  }

  return std::pair(Reg, 0);
}

IntrinsicLaneMaskAnalyzer::IntrinsicLaneMaskAnalyzer(MachineFunction &MF)
    : MRI(MF.getRegInfo()) {
  initLaneMaskIntrinsics(MF);
}

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `std::pair`, `IntrinsicLaneMaskAnalyzer::IntrinsicLaneMaskAnalyzer`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`std::pair`, `IntrinsicLaneMaskAnalyzer::IntrinsicLaneMaskAnalyzer`。

### Lines 86-106: Implements IntrinsicLaneMaskAnalyzer::isS32S64LaneMask
```cpp
bool IntrinsicLaneMaskAnalyzer::isS32S64LaneMask(Register Reg) const {
  return S32S64LaneMask.contains(Reg);
}

void IntrinsicLaneMaskAnalyzer::initLaneMaskIntrinsics(MachineFunction &MF) {
  for (auto &MBB : MF) {
    for (auto &MI : MBB) {
      GIntrinsic *GI = dyn_cast<GIntrinsic>(&MI);
      if (GI && GI->is(Intrinsic::amdgcn_if_break)) {
        S32S64LaneMask.insert(MI.getOperand(3).getReg());
        S32S64LaneMask.insert(MI.getOperand(0).getReg());
      }

      if (MI.getOpcode() == AMDGPU::SI_IF ||
          MI.getOpcode() == AMDGPU::SI_ELSE) {
        S32S64LaneMask.insert(MI.getOperand(0).getReg());
      }
    }
  }
}

```
**EN:** This section contains concrete logic for IntrinsicLaneMaskAnalyzer::isS32S64LaneMask. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `IntrinsicLaneMaskAnalyzer::isS32S64LaneMask`, `IntrinsicLaneMaskAnalyzer::initLaneMaskIntrinsics`.
**CN:** 本节包含与 IntrinsicLaneMaskAnalyzer::isS32S64LaneMask 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`IntrinsicLaneMaskAnalyzer::isS32S64LaneMask`, `IntrinsicLaneMaskAnalyzer::initLaneMaskIntrinsics`。

### Lines 107-123: Defines getReadAnyLaneSplitTy
```cpp
static LLT getReadAnyLaneSplitTy(LLT Ty) {
  if (Ty.isVector()) {
    LLT ElTy = Ty.getElementType();
    if (ElTy.getSizeInBits() == 16)
      return LLT::fixed_vector(2, ElTy);
    // S32, S64 or pointer
    return ElTy;
  }

  // Large scalars and 64-bit pointers
  return LLT::scalar(32);
}

template <typename ReadLaneFnTy>
static Register buildReadLane(MachineIRBuilder &, Register,
                              const RegisterBankInfo &, ReadLaneFnTy);

```
**EN:** This section contains concrete logic for getReadAnyLaneSplitTy. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `LLT::fixed_vector`, `LLT::scalar`.
**CN:** 本节包含与 getReadAnyLaneSplitTy 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`LLT::fixed_vector`, `LLT::scalar`。

### Lines 124-146: Defines unmergeReadAnyLane
```cpp
template <typename ReadLaneFnTy>
static void
unmergeReadAnyLane(MachineIRBuilder &B, SmallVectorImpl<Register> &SgprDstParts,
                   LLT UnmergeTy, Register VgprSrc, const RegisterBankInfo &RBI,
                   ReadLaneFnTy BuildRL) {
  const RegisterBank *VgprRB = &RBI.getRegBank(AMDGPU::VGPRRegBankID);
  auto Unmerge = B.buildUnmerge({VgprRB, UnmergeTy}, VgprSrc);
  for (unsigned i = 0; i < Unmerge->getNumOperands() - 1; ++i) {
    SgprDstParts.push_back(buildReadLane(B, Unmerge.getReg(i), RBI, BuildRL));
  }
}

template <typename ReadLaneFnTy>
static Register buildReadLane(MachineIRBuilder &B, Register VgprSrc,
                              const RegisterBankInfo &RBI,
                              ReadLaneFnTy BuildRL) {
  LLT Ty = B.getMRI()->getType(VgprSrc);
  const RegisterBank *SgprRB = &RBI.getRegBank(AMDGPU::SGPRRegBankID);
  if (Ty.getSizeInBits() == 32) {
    Register SgprDst = B.getMRI()->createVirtualRegister({SgprRB, Ty});
    return BuildRL(B, SgprDst, VgprSrc).getReg(0);
  }

```
**EN:** This section contains concrete logic for unmergeReadAnyLane. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 unmergeReadAnyLane 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 147-170: Defines unmergeReadAnyLane
```cpp
  SmallVector<Register, 8> SgprDstParts;
  unmergeReadAnyLane(B, SgprDstParts, getReadAnyLaneSplitTy(Ty), VgprSrc, RBI,
                     BuildRL);

  return B.buildMergeLikeInstr({SgprRB, Ty}, SgprDstParts).getReg(0);
}

template <typename ReadLaneFnTy>
static void buildReadLane(MachineIRBuilder &B, Register SgprDst,
                          Register VgprSrc, const RegisterBankInfo &RBI,
                          ReadLaneFnTy BuildReadLane) {
  LLT Ty = B.getMRI()->getType(VgprSrc);
  if (Ty.getSizeInBits() == 32) {
    BuildReadLane(B, SgprDst, VgprSrc);
    return;
  }

  SmallVector<Register, 8> SgprDstParts;
  unmergeReadAnyLane(B, SgprDstParts, getReadAnyLaneSplitTy(Ty), VgprSrc, RBI,
                     BuildReadLane);

  B.buildMergeLikeInstr(SgprDst, SgprDstParts).getReg(0);
}

```
**EN:** This section contains concrete logic for unmergeReadAnyLane. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 unmergeReadAnyLane 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 171-188: Implements AMDGPU::buildReadAnyLane
```cpp
void AMDGPU::buildReadAnyLane(MachineIRBuilder &B, Register SgprDst,
                              Register VgprSrc, const RegisterBankInfo &RBI) {
  return buildReadLane(
      B, SgprDst, VgprSrc, RBI,
      [](MachineIRBuilder &B, Register SgprDst, Register VgprSrc) {
        return B.buildInstr(AMDGPU::G_AMDGPU_READANYLANE, {SgprDst}, {VgprSrc});
      });
}

void AMDGPU::buildReadFirstLane(MachineIRBuilder &B, Register SgprDst,
                                Register VgprSrc, const RegisterBankInfo &RBI) {
  return buildReadLane(
      B, SgprDst, VgprSrc, RBI,
      [](MachineIRBuilder &B, Register SgprDst, Register VgprSrc) {
        return B.buildIntrinsic(Intrinsic::amdgcn_readfirstlane, SgprDst)
            .addReg(VgprSrc);
      });
}
```
**EN:** This section contains concrete logic for AMDGPU::buildReadAnyLane. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPU::buildReadAnyLane`, `AMDGPU::buildReadFirstLane`.
**CN:** 本节包含与 AMDGPU::buildReadAnyLane 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPU::buildReadAnyLane`, `AMDGPU::buildReadFirstLane`。

## Key Concepts / 关键概念
- **Language / 语言**: C++ source
- **Primary symbols / 主要符号**: `AMDGPU::getBaseWithConstantOffset`, `std::pair`, `IntrinsicLaneMaskAnalyzer::IntrinsicLaneMaskAnalyzer`, `IntrinsicLaneMaskAnalyzer::isS32S64LaneMask`, `IntrinsicLaneMaskAnalyzer::initLaneMaskIntrinsics`, `LLT::fixed_vector`
- **Main themes / 核心主题**: register management / 寄存器管理; instruction semantics / 指令语义; machine-function state / MachineFunction 状态; GlobalISel support / GlobalISel 支持
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"AMDGPUGlobalISelUtils.h"`
- `"AMDGPURegisterBankInfo.h"`
- `"MCTargetDesc/AMDGPUMCTargetDesc.h"`
- `"llvm/ADT/DenseSet.h"`
- `"llvm/CodeGen/GlobalISel/GISelValueTracking.h"`
- `"llvm/CodeGen/GlobalISel/GenericMachineInstrs.h"`
- `"llvm/CodeGen/GlobalISel/MIPatternMatch.h"`
- `"llvm/CodeGen/GlobalISel/MachineIRBuilder.h"`
- `"llvm/CodeGenTypes/LowLevelType.h"`
- `"llvm/IR/Constants.h"`
- `"llvm/IR/IntrinsicsAMDGPU.h"`
