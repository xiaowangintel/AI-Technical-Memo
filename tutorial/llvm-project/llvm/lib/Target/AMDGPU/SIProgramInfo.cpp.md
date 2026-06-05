# SIProgramInfo.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/SIProgramInfo.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This source file implements SIProgramInfo for the LLVM AMDGPU backend. It contains target-specific logic used during analysis, lowering, code generation, or pass execution. / 该源文件实现 LLVM AMDGPU 后端中的 SIProgramInfo 相关功能。它包含分析、降低、代码生成或 Pass 执行过程中使用的目标专用逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-31: File banner, includes, and setup
```cpp
//===-- SIProgramInfo.cpp ----------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
/// \file
///
/// The SIProgramInfo tracks resource usage and hardware flags for kernels and
/// entry functions.
//
//===----------------------------------------------------------------------===//
//

#include "SIProgramInfo.h"
#include "GCNSubtarget.h"
#include "SIDefines.h"
#include "Utils/AMDGPUBaseInfo.h"
#include "llvm/MC/MCExpr.h"

using namespace llvm;

void SIProgramInfo::reset(const MachineFunction &MF) {
  MCContext &Ctx = MF.getContext();

  const MCExpr *ZeroExpr = MCConstantExpr::create(0, Ctx);

  CodeSizeInBytes.reset();

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation. Main symbols: `SIProgramInfo::reset`, `MCConstantExpr::create`.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。 主要符号：`SIProgramInfo::reset`, `MCConstantExpr::create`。

### Lines 32-62: Implementation details and local logic
```cpp
  VGPRBlocks = ZeroExpr;
  SGPRBlocks = ZeroExpr;
  Priority = 0;
  FloatMode = 0;
  Priv = 0;
  DX10Clamp = 0;
  DebugMode = 0;
  IEEEMode = 0;
  WgpMode = 0;
  MemOrdered = 0;
  FwdProgress = 0;
  RrWgMode = 0;
  ScratchSize = ZeroExpr;

  LDSBlocks = 0;
  ScratchBlocks = ZeroExpr;

  ScratchEnable = ZeroExpr;
  UserSGPR = 0;
  TrapHandlerEnable = 0;
  TGIdXEnable = 0;
  TGIdYEnable = 0;
  TGIdZEnable = 0;
  TGSizeEnable = 0;
  TIdIGCompCount = 0;
  EXCPEnMSB = 0;
  LdsSize = 0;
  EXCPEnable = 0;

  ComputePGMRSrc3 = ZeroExpr;

```
**EN:** This section advances the file implementation by combining declarations, control flow, and helper operations that support the AMDGPU backend.
**CN:** 本节通过组合声明、控制流与辅助操作来推进文件实现，支撑 AMDGPU 后端的整体功能。

### Lines 63-96: Conditional logic and checks
```cpp
  NumVGPR = ZeroExpr;
  NumArchVGPR = ZeroExpr;
  NumAccVGPR = ZeroExpr;
  AccumOffset = ZeroExpr;
  TgSplit = 0;
  NumSGPR = ZeroExpr;
  SGPRSpill = 0;
  VGPRSpill = 0;
  LDSSize = 0;
  FlatUsed = ZeroExpr;

  NumSGPRsForWavesPerEU = ZeroExpr;
  NumVGPRsForWavesPerEU = ZeroExpr;
  NamedBarCnt = ZeroExpr;
  Occupancy = ZeroExpr;
  DynamicCallStack = ZeroExpr;
  VCCUsed = ZeroExpr;
}

static uint64_t getComputePGMRSrc1Reg(const SIProgramInfo &ProgInfo,
                                      const GCNSubtarget &ST) {
  uint64_t Reg = S_00B848_PRIORITY(ProgInfo.Priority) |
                 S_00B848_FLOAT_MODE(ProgInfo.FloatMode) |
                 S_00B848_PRIV(ProgInfo.Priv) |
                 S_00B848_DEBUG_MODE(ProgInfo.DebugMode) |
                 S_00B848_WGP_MODE(ProgInfo.WgpMode) |
                 S_00B848_MEM_ORDERED(ProgInfo.MemOrdered) |
                 S_00B848_FWD_PROGRESS(ProgInfo.FwdProgress);

  if (ST.hasFeature(AMDGPU::FeatureDX10ClampAndIEEEMode)) {
    Reg |= S_00B848_DX10_CLAMP(ProgInfo.DX10Clamp);
    Reg |= S_00B848_IEEE_MODE(ProgInfo.IEEEMode);
  }

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 97-117: Defines getPGMRSrc1Reg
```cpp
  if (ST.hasRrWGMode())
    Reg |= S_00B848_RR_WG_MODE(ProgInfo.RrWgMode);

  return Reg;
}

static uint64_t getPGMRSrc1Reg(const SIProgramInfo &ProgInfo,
                               CallingConv::ID CC, const GCNSubtarget &ST) {
  uint64_t Reg = S_00B848_PRIORITY(ProgInfo.Priority) |
                 S_00B848_FLOAT_MODE(ProgInfo.FloatMode) |
                 S_00B848_PRIV(ProgInfo.Priv) |
                 S_00B848_DEBUG_MODE(ProgInfo.DebugMode);

  if (ST.hasFeature(AMDGPU::FeatureDX10ClampAndIEEEMode)) {
    Reg |= S_00B848_DX10_CLAMP(ProgInfo.DX10Clamp);
    Reg |= S_00B848_IEEE_MODE(ProgInfo.IEEEMode);
  }

  if (ST.hasRrWGMode())
    Reg |= S_00B848_RR_WG_MODE(ProgInfo.RrWgMode);

```
**EN:** This section contains concrete logic for getPGMRSrc1Reg. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 getPGMRSrc1Reg 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 118-151: Switch-based control flow
```cpp
  switch (CC) {
  case CallingConv::AMDGPU_PS:
    Reg |= S_00B028_MEM_ORDERED(ProgInfo.MemOrdered);
    break;
  case CallingConv::AMDGPU_VS:
    Reg |= S_00B128_MEM_ORDERED(ProgInfo.MemOrdered);
    break;
  case CallingConv::AMDGPU_GS:
    Reg |= S_00B228_WGP_MODE(ProgInfo.WgpMode) |
           S_00B228_MEM_ORDERED(ProgInfo.MemOrdered);
    break;
  case CallingConv::AMDGPU_HS:
    Reg |= S_00B428_WGP_MODE(ProgInfo.WgpMode) |
           S_00B428_MEM_ORDERED(ProgInfo.MemOrdered);
    break;
  default:
    break;
  }
  return Reg;
}

static uint64_t getComputePGMRSrc2Reg(const GCNSubtarget &ST,
                                      const SIProgramInfo &ProgInfo) {
  uint64_t MaxNumUserSGRPs = AMDGPU::getMaxNumUserSGPRs(ST);
  uint64_t Reg = 0;
  if (MaxNumUserSGRPs == 32) {
    Reg = S_00B84C_USER_SGPR_GFX1250(ProgInfo.UserSGPR);
  } else if (MaxNumUserSGRPs == 16) {
    Reg = (S_00B84C_USER_SGPR(ProgInfo.UserSGPR) |
           S_00B84C_TRAP_HANDLER(ProgInfo.TrapHandlerEnable));
  } else {
    llvm_unreachable("max Number of User SGPRs are either 32 or 16");
  }

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `AMDGPU::getMaxNumUserSGPRs`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`AMDGPU::getMaxNumUserSGPRs`。

### Lines 152-176: Defines S_00B84C_TGID_X_EN
```cpp
  Reg |= S_00B84C_TGID_X_EN(ProgInfo.TGIdXEnable) |
         S_00B84C_TGID_Y_EN(ProgInfo.TGIdYEnable) |
         S_00B84C_TGID_Z_EN(ProgInfo.TGIdZEnable) |
         S_00B84C_TG_SIZE_EN(ProgInfo.TGSizeEnable) |
         S_00B84C_TIDIG_COMP_CNT(ProgInfo.TIdIGCompCount) |
         S_00B84C_EXCP_EN_MSB(ProgInfo.EXCPEnMSB) |
         S_00B84C_LDS_SIZE(ProgInfo.LdsSize) |
         S_00B84C_EXCP_EN(ProgInfo.EXCPEnable);

  return Reg;
}

static const MCExpr *MaskShift(const MCExpr *Val, uint32_t Mask, uint32_t Shift,
                               MCContext &Ctx) {
  if (Mask) {
    const MCExpr *MaskExpr = MCConstantExpr::create(Mask, Ctx);
    Val = MCBinaryExpr::createAnd(Val, MaskExpr, Ctx);
  }
  if (Shift) {
    const MCExpr *ShiftExpr = MCConstantExpr::create(Shift, Ctx);
    Val = MCBinaryExpr::createShl(Val, ShiftExpr, Ctx);
  }
  return Val;
}

```
**EN:** This section contains concrete logic for S_00B84C_TGID_X_EN. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `MCConstantExpr::create`, `MCBinaryExpr::createAnd`, `MCBinaryExpr::createShl`.
**CN:** 本节包含与 S_00B84C_TGID_X_EN 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`MCConstantExpr::create`, `MCBinaryExpr::createAnd`, `MCBinaryExpr::createShl`。

### Lines 177-208: Implements SIProgramInfo::getComputePGMRSrc1
```cpp
const MCExpr *SIProgramInfo::getComputePGMRSrc1(const GCNSubtarget &ST,
                                                MCContext &Ctx) const {
  uint64_t Reg = getComputePGMRSrc1Reg(*this, ST);
  const MCExpr *RegExpr = MCConstantExpr::create(Reg, Ctx);
  const MCExpr *Res = MCBinaryExpr::createOr(
      MaskShift(VGPRBlocks, /*Mask=*/0x3F, /*Shift=*/0, Ctx),
      MaskShift(SGPRBlocks, /*Mask=*/0xF, /*Shift=*/6, Ctx), Ctx);
  return MCBinaryExpr::createOr(RegExpr, Res, Ctx);
}

const MCExpr *SIProgramInfo::getPGMRSrc1(CallingConv::ID CC,
                                         const GCNSubtarget &ST,
                                         MCContext &Ctx) const {
  if (AMDGPU::isCompute(CC)) {
    return getComputePGMRSrc1(ST, Ctx);
  }

  uint64_t Reg = getPGMRSrc1Reg(*this, CC, ST);
  const MCExpr *RegExpr = MCConstantExpr::create(Reg, Ctx);
  const MCExpr *Res = MCBinaryExpr::createOr(
      MaskShift(VGPRBlocks, /*Mask=*/0x3F, /*Shift=*/0, Ctx),
      MaskShift(SGPRBlocks, /*Mask=*/0xF, /*Shift=*/6, Ctx), Ctx);
  return MCBinaryExpr::createOr(RegExpr, Res, Ctx);
}

const MCExpr *SIProgramInfo::getComputePGMRSrc2(const GCNSubtarget &ST,
                                                MCContext &Ctx) const {
  uint64_t Reg = getComputePGMRSrc2Reg(ST, *this);
  const MCExpr *RegExpr = MCConstantExpr::create(Reg, Ctx);
  return MCBinaryExpr::createOr(ScratchEnable, RegExpr, Ctx);
}

```
**EN:** This section contains concrete logic for SIProgramInfo::getComputePGMRSrc1. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `SIProgramInfo::getComputePGMRSrc1`, `MCConstantExpr::create`, `MCBinaryExpr::createOr`.
**CN:** 本节包含与 SIProgramInfo::getComputePGMRSrc1 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`SIProgramInfo::getComputePGMRSrc1`, `MCConstantExpr::create`, `MCBinaryExpr::createOr`。

### Lines 209-241: Implements SIProgramInfo::getPGMRSrc2
```cpp
const MCExpr *SIProgramInfo::getPGMRSrc2(CallingConv::ID CC,
                                         const GCNSubtarget &ST,
                                         MCContext &Ctx) const {
  if (AMDGPU::isCompute(CC))
    return getComputePGMRSrc2(ST, Ctx);

  return MCConstantExpr::create(0, Ctx);
}

uint64_t SIProgramInfo::getFunctionCodeSize(const MachineFunction &MF,
                                            bool IsLowerBound) {
  if (!IsLowerBound && CodeSizeInBytes.has_value())
    return *CodeSizeInBytes;

  const GCNSubtarget &STM = MF.getSubtarget<GCNSubtarget>();
  const SIInstrInfo *TII = STM.getInstrInfo();

  uint64_t CodeSize = 0;

  for (const MachineBasicBlock &MBB : MF) {
    // The amount of padding to align code can be both underestimated and
    // overestimated. In case of inline asm used getInstSizeInBytes() will
    // return a maximum size of a single instruction, where the real size may
    // differ. At this point CodeSize may be already off.
    if (!IsLowerBound)
      CodeSize = alignTo(CodeSize, MBB.getAlignment());

    for (const MachineInstr &MI : MBB) {
      // TODO: CodeSize should account for multiple functions.

      if (MI.isMetaInstruction())
        continue;

```
**EN:** This section contains concrete logic for SIProgramInfo::getPGMRSrc2. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `SIProgramInfo::getPGMRSrc2`, `AMDGPU::isCompute`, `MCConstantExpr::create`.
**CN:** 本节包含与 SIProgramInfo::getPGMRSrc2 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`SIProgramInfo::getPGMRSrc2`, `AMDGPU::isCompute`, `MCConstantExpr::create`。

### Lines 242-253: Conditional logic and checks
```cpp
      // We cannot properly estimate inline asm size. It can be as small as zero
      // if that is just a comment.
      if (IsLowerBound && MI.isInlineAsm())
        continue;

      CodeSize += TII->getInstSizeInBytes(MI);
    }
  }

  CodeSizeInBytes = CodeSize;
  return CodeSize;
}
```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

## Key Concepts / 关键概念
- **Language / 语言**: C++ source
- **Primary symbols / 主要符号**: `SIProgramInfo::reset`, `MCConstantExpr::create`, `AMDGPU::getMaxNumUserSGPRs`, `MCBinaryExpr::createAnd`, `MCBinaryExpr::createShl`, `SIProgramInfo::getComputePGMRSrc1`
- **Main themes / 核心主题**: subtarget modeling / 子目标建模; feature description / 特性描述; machine-function state / MachineFunction 状态; hardware resource modeling / 硬件资源建模
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"SIProgramInfo.h"`
- `"GCNSubtarget.h"`
- `"SIDefines.h"`
- `"Utils/AMDGPUBaseInfo.h"`
- `"llvm/MC/MCExpr.h"`
