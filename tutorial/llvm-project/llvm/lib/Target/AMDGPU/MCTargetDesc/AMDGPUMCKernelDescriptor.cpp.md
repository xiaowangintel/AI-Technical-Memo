# AMDGPUMCKernelDescriptor.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/MCTargetDesc/AMDGPUMCKernelDescriptor.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This source file implements AMDGPUMCKernelDescriptor for the LLVM MC target description layer. It contains target-specific logic used during analysis, lowering, code generation, or pass execution. / 该源文件实现 LLVM MC 目标描述层中的 AMDGPUMCKernelDescriptor 相关功能。它包含分析、降低、代码生成或 Pass 执行过程中使用的目标专用逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-24: File banner, includes, and setup
```cpp
//===--- AMDHSAKernelDescriptor.h -----------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "AMDGPUMCKernelDescriptor.h"
#include "AMDGPUMCTargetDesc.h"
#include "Utils/AMDGPUBaseInfo.h"
#include "llvm/MC/MCContext.h"
#include "llvm/MC/MCExpr.h"
#include "llvm/MC/MCSubtargetInfo.h"
#include "llvm/TargetParser/TargetParser.h"

using namespace llvm;
using namespace llvm::AMDGPU;

MCKernelDescriptor
MCKernelDescriptor::getDefaultAmdhsaKernelDescriptor(const MCSubtargetInfo *STI,
                                                     MCContext &Ctx) {
  IsaVersion Version = getIsaVersion(STI->getCPU());

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation. Main symbols: `MCKernelDescriptor::getDefaultAmdhsaKernelDescriptor`.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。 主要符号：`MCKernelDescriptor::getDefaultAmdhsaKernelDescriptor`。

### Lines 25-37: Implements MCConstantExpr::create
```cpp
  MCKernelDescriptor KD;
  const MCExpr *ZeroMCExpr = MCConstantExpr::create(0, Ctx);
  const MCExpr *OneMCExpr = MCConstantExpr::create(1, Ctx);

  KD.group_segment_fixed_size = ZeroMCExpr;
  KD.private_segment_fixed_size = ZeroMCExpr;
  KD.compute_pgm_rsrc1 = ZeroMCExpr;
  KD.compute_pgm_rsrc2 = ZeroMCExpr;
  KD.compute_pgm_rsrc3 = ZeroMCExpr;
  KD.kernarg_size = ZeroMCExpr;
  KD.kernel_code_properties = ZeroMCExpr;
  KD.kernarg_preload = ZeroMCExpr;

```
**EN:** This section contains concrete logic for MCConstantExpr::create. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `MCConstantExpr::create`.
**CN:** 本节包含与 MCConstantExpr::create 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`MCConstantExpr::create`。

### Lines 38-61: Implements MCKernelDescriptor::bits_set
```cpp
  MCKernelDescriptor::bits_set(
      KD.compute_pgm_rsrc1,
      MCConstantExpr::create(amdhsa::FLOAT_DENORM_MODE_FLUSH_NONE, Ctx),
      amdhsa::COMPUTE_PGM_RSRC1_FLOAT_DENORM_MODE_16_64_SHIFT,
      amdhsa::COMPUTE_PGM_RSRC1_FLOAT_DENORM_MODE_16_64, Ctx);
  if (STI->hasFeature(AMDGPU::FeatureDX10ClampAndIEEEMode)) {
    MCKernelDescriptor::bits_set(
        KD.compute_pgm_rsrc1, OneMCExpr,
        amdhsa::COMPUTE_PGM_RSRC1_GFX6_GFX11_ENABLE_DX10_CLAMP_SHIFT,
        amdhsa::COMPUTE_PGM_RSRC1_GFX6_GFX11_ENABLE_DX10_CLAMP, Ctx);
    MCKernelDescriptor::bits_set(
        KD.compute_pgm_rsrc1, OneMCExpr,
        amdhsa::COMPUTE_PGM_RSRC1_GFX6_GFX11_ENABLE_IEEE_MODE_SHIFT,
        amdhsa::COMPUTE_PGM_RSRC1_GFX6_GFX11_ENABLE_IEEE_MODE, Ctx);
  }
  MCKernelDescriptor::bits_set(
      KD.compute_pgm_rsrc2, OneMCExpr,
      amdhsa::COMPUTE_PGM_RSRC2_ENABLE_SGPR_WORKGROUP_ID_X_SHIFT,
      amdhsa::COMPUTE_PGM_RSRC2_ENABLE_SGPR_WORKGROUP_ID_X, Ctx);
  if (Version.Major >= 10) {
    if (STI->getFeatureBits().test(FeatureWavefrontSize32))
      MCKernelDescriptor::bits_set(
          KD.kernel_code_properties, OneMCExpr,
          amdhsa::KERNEL_CODE_PROPERTY_ENABLE_WAVEFRONT_SIZE32_SHIFT,
```
**EN:** This section contains concrete logic for MCKernelDescriptor::bits_set. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `MCKernelDescriptor::bits_set`, `MCConstantExpr::create`.
**CN:** 本节包含与 MCKernelDescriptor::bits_set 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`MCKernelDescriptor::bits_set`, `MCConstantExpr::create`。

### Lines 62-85: Conditional logic and checks
```cpp
          amdhsa::KERNEL_CODE_PROPERTY_ENABLE_WAVEFRONT_SIZE32, Ctx);
    if (!STI->getFeatureBits().test(FeatureCuMode))
      MCKernelDescriptor::bits_set(
          KD.compute_pgm_rsrc1, OneMCExpr,
          amdhsa::COMPUTE_PGM_RSRC1_GFX10_PLUS_WGP_MODE_SHIFT,
          amdhsa::COMPUTE_PGM_RSRC1_GFX10_PLUS_WGP_MODE, Ctx);

    MCKernelDescriptor::bits_set(
        KD.compute_pgm_rsrc1, OneMCExpr,
        amdhsa::COMPUTE_PGM_RSRC1_GFX10_PLUS_MEM_ORDERED_SHIFT,
        amdhsa::COMPUTE_PGM_RSRC1_GFX10_PLUS_MEM_ORDERED, Ctx);

    MCKernelDescriptor::bits_set(
        KD.compute_pgm_rsrc1, OneMCExpr,
        amdhsa::COMPUTE_PGM_RSRC1_GFX10_PLUS_FWD_PROGRESS_SHIFT,
        amdhsa::COMPUTE_PGM_RSRC1_GFX10_PLUS_FWD_PROGRESS, Ctx);
  }
  if (AMDGPU::isGFX90A(*STI) && STI->getFeatureBits().test(FeatureTgSplit))
    MCKernelDescriptor::bits_set(
        KD.compute_pgm_rsrc3, OneMCExpr,
        amdhsa::COMPUTE_PGM_RSRC3_GFX90A_TG_SPLIT_SHIFT,
        amdhsa::COMPUTE_PGM_RSRC3_GFX90A_TG_SPLIT, Ctx);
  return KD;
}
```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `MCKernelDescriptor::bits_set`, `AMDGPU::isGFX90A`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`MCKernelDescriptor::bits_set`, `AMDGPU::isGFX90A`。

### Lines 86-103: Implements MCKernelDescriptor::bits_set
```cpp

void MCKernelDescriptor::bits_set(const MCExpr *&Dst, const MCExpr *Value,
                                  uint32_t Shift, uint32_t Mask,
                                  MCContext &Ctx) {
  const auto *Sft = MCConstantExpr::create(Shift, Ctx);
  const auto *Msk = MCConstantExpr::create(Mask, Ctx);
  Dst = MCBinaryExpr::createAnd(Dst, MCUnaryExpr::createNot(Msk, Ctx), Ctx);
  Dst = MCBinaryExpr::createOr(Dst, MCBinaryExpr::createShl(Value, Sft, Ctx),
                               Ctx);
}

const MCExpr *MCKernelDescriptor::bits_get(const MCExpr *Src, uint32_t Shift,
                                           uint32_t Mask, MCContext &Ctx) {
  const auto *Sft = MCConstantExpr::create(Shift, Ctx);
  const auto *Msk = MCConstantExpr::create(Mask, Ctx);
  return MCBinaryExpr::createLShr(MCBinaryExpr::createAnd(Src, Msk, Ctx), Sft,
                                  Ctx);
}
```
**EN:** This section contains concrete logic for MCKernelDescriptor::bits_set. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `MCKernelDescriptor::bits_set`, `MCConstantExpr::create`, `MCBinaryExpr::createAnd`.
**CN:** 本节包含与 MCKernelDescriptor::bits_set 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`MCKernelDescriptor::bits_set`, `MCConstantExpr::create`, `MCBinaryExpr::createAnd`。

## Key Concepts / 关键概念
- **Language / 语言**: C++ source
- **Primary symbols / 主要符号**: `MCKernelDescriptor::getDefaultAmdhsaKernelDescriptor`, `MCConstantExpr::create`, `MCKernelDescriptor::bits_set`, `AMDGPU::isGFX90A`, `MCBinaryExpr::createAnd`, `MCUnaryExpr::createNot`
- **Main themes / 核心主题**: subtarget modeling / 子目标建模; feature description / 特性描述
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"AMDGPUMCKernelDescriptor.h"`
- `"AMDGPUMCTargetDesc.h"`
- `"Utils/AMDGPUBaseInfo.h"`
- `"llvm/MC/MCContext.h"`
- `"llvm/MC/MCExpr.h"`
- `"llvm/MC/MCSubtargetInfo.h"`
- `"llvm/TargetParser/TargetParser.h"`
