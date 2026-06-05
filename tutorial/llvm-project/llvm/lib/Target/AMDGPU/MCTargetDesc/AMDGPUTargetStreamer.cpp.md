# AMDGPUTargetStreamer.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/MCTargetDesc/AMDGPUTargetStreamer.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This source file implements AMDGPUTargetStreamer for the LLVM MC target description layer. It contains target-specific logic used during analysis, lowering, code generation, or pass execution. / 该源文件实现 LLVM MC 目标描述层中的 AMDGPUTargetStreamer 相关功能。它包含分析、降低、代码生成或 Pass 执行过程中使用的目标专用逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-42: File banner, includes, and setup
```cpp
//===-- AMDGPUTargetStreamer.cpp - Mips Target Streamer Methods -----------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file provides AMDGPU specific target streamer methods.
//
//===----------------------------------------------------------------------===//

#include "AMDGPUTargetStreamer.h"
#include "AMDGPUMCExpr.h"
#include "AMDGPUMCKernelDescriptor.h"
#include "AMDGPUMCTargetDesc.h"
#include "AMDGPUPTNote.h"
#include "Utils/AMDGPUBaseInfo.h"
#include "Utils/AMDKernelCodeTUtils.h"
#include "llvm/BinaryFormat/AMDGPUMetadataVerifier.h"
#include "llvm/BinaryFormat/ELF.h"
#include "llvm/MC/MCAsmInfo.h"
#include "llvm/MC/MCAssembler.h"
#include "llvm/MC/MCContext.h"
#include "llvm/MC/MCELFObjectWriter.h"
#include "llvm/MC/MCELFStreamer.h"
#include "llvm/MC/MCSubtargetInfo.h"
#include "llvm/MC/StringTableBuilder.h"
#include "llvm/Support/AMDGPUMetadata.h"
#include "llvm/Support/AMDGPUObjLinkingInfo.h"
#include "llvm/Support/AMDHSAKernelDescriptor.h"
#include "llvm/Support/CommandLine.h"
#include "llvm/Support/FormattedStream.h"
#include "llvm/TargetParser/TargetParser.h"

using namespace llvm;
using namespace llvm::AMDGPU;

//===----------------------------------------------------------------------===//
// AMDGPUTargetStreamer
//===----------------------------------------------------------------------===//

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。

### Lines 43-86: Implements AMDGPUTargetStreamer::EmitHSAMetadataV3
```cpp
static cl::opt<unsigned>
    ForceGenericVersion("amdgpu-force-generic-version",
                        cl::desc("Force a specific generic_v<N> flag to be "
                                 "added. For testing purposes only."),
                        cl::ReallyHidden, cl::init(0));

bool AMDGPUTargetStreamer::EmitHSAMetadataV3(StringRef HSAMetadataString) {
  msgpack::Document HSAMetadataDoc;
  if (!HSAMetadataDoc.fromYAML(HSAMetadataString))
    return false;
  return EmitHSAMetadata(HSAMetadataDoc, false);
}

StringRef AMDGPUTargetStreamer::getArchNameFromElfMach(unsigned ElfMach) {
  AMDGPU::GPUKind AK;

  // clang-format off
  switch (ElfMach) {
  case ELF::EF_AMDGPU_MACH_R600_R600:      AK = GK_R600;    break;
  case ELF::EF_AMDGPU_MACH_R600_R630:      AK = GK_R630;    break;
  case ELF::EF_AMDGPU_MACH_R600_RS880:     AK = GK_RS880;   break;
  case ELF::EF_AMDGPU_MACH_R600_RV670:     AK = GK_RV670;   break;
  case ELF::EF_AMDGPU_MACH_R600_RV710:     AK = GK_RV710;   break;
  case ELF::EF_AMDGPU_MACH_R600_RV730:     AK = GK_RV730;   break;
  case ELF::EF_AMDGPU_MACH_R600_RV770:     AK = GK_RV770;   break;
  case ELF::EF_AMDGPU_MACH_R600_CEDAR:     AK = GK_CEDAR;   break;
  case ELF::EF_AMDGPU_MACH_R600_CYPRESS:   AK = GK_CYPRESS; break;
  case ELF::EF_AMDGPU_MACH_R600_JUNIPER:   AK = GK_JUNIPER; break;
  case ELF::EF_AMDGPU_MACH_R600_REDWOOD:   AK = GK_REDWOOD; break;
  case ELF::EF_AMDGPU_MACH_R600_SUMO:      AK = GK_SUMO;    break;
  case ELF::EF_AMDGPU_MACH_R600_BARTS:     AK = GK_BARTS;   break;
  case ELF::EF_AMDGPU_MACH_R600_CAICOS:    AK = GK_CAICOS;  break;
  case ELF::EF_AMDGPU_MACH_R600_CAYMAN:    AK = GK_CAYMAN;  break;
  case ELF::EF_AMDGPU_MACH_R600_TURKS:     AK = GK_TURKS;   break;
  case ELF::EF_AMDGPU_MACH_AMDGCN_GFX600:  AK = GK_GFX600;  break;
  case ELF::EF_AMDGPU_MACH_AMDGCN_GFX601:  AK = GK_GFX601;  break;
  case ELF::EF_AMDGPU_MACH_AMDGCN_GFX602:  AK = GK_GFX602;  break;
  case ELF::EF_AMDGPU_MACH_AMDGCN_GFX700:  AK = GK_GFX700;  break;
  case ELF::EF_AMDGPU_MACH_AMDGCN_GFX701:  AK = GK_GFX701;  break;
  case ELF::EF_AMDGPU_MACH_AMDGCN_GFX702:  AK = GK_GFX702;  break;
  case ELF::EF_AMDGPU_MACH_AMDGCN_GFX703:  AK = GK_GFX703;  break;
  case ELF::EF_AMDGPU_MACH_AMDGCN_GFX704:  AK = GK_GFX704;  break;
  case ELF::EF_AMDGPU_MACH_AMDGCN_GFX705:  AK = GK_GFX705;  break;
  case ELF::EF_AMDGPU_MACH_AMDGCN_GFX801:  AK = GK_GFX801;  break;
```
**EN:** This section contains concrete logic for AMDGPUTargetStreamer::EmitHSAMetadataV3. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `cl::desc`, `cl::init`, `AMDGPUTargetStreamer::EmitHSAMetadataV3`.
**CN:** 本节包含与 AMDGPUTargetStreamer::EmitHSAMetadataV3 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`cl::desc`, `cl::init`, `AMDGPUTargetStreamer::EmitHSAMetadataV3`。

### Lines 87-130: Implementation details and local logic
```cpp
  case ELF::EF_AMDGPU_MACH_AMDGCN_GFX802:  AK = GK_GFX802;  break;
  case ELF::EF_AMDGPU_MACH_AMDGCN_GFX803:  AK = GK_GFX803;  break;
  case ELF::EF_AMDGPU_MACH_AMDGCN_GFX805:  AK = GK_GFX805;  break;
  case ELF::EF_AMDGPU_MACH_AMDGCN_GFX810:  AK = GK_GFX810;  break;
  case ELF::EF_AMDGPU_MACH_AMDGCN_GFX900:  AK = GK_GFX900;  break;
  case ELF::EF_AMDGPU_MACH_AMDGCN_GFX902:  AK = GK_GFX902;  break;
  case ELF::EF_AMDGPU_MACH_AMDGCN_GFX904:  AK = GK_GFX904;  break;
  case ELF::EF_AMDGPU_MACH_AMDGCN_GFX906:  AK = GK_GFX906;  break;
  case ELF::EF_AMDGPU_MACH_AMDGCN_GFX908:  AK = GK_GFX908;  break;
  case ELF::EF_AMDGPU_MACH_AMDGCN_GFX909:  AK = GK_GFX909;  break;
  case ELF::EF_AMDGPU_MACH_AMDGCN_GFX90A:  AK = GK_GFX90A;  break;
  case ELF::EF_AMDGPU_MACH_AMDGCN_GFX90C:  AK = GK_GFX90C;  break;
  case ELF::EF_AMDGPU_MACH_AMDGCN_GFX942:  AK = GK_GFX942;  break;
  case ELF::EF_AMDGPU_MACH_AMDGCN_GFX950:  AK = GK_GFX950;  break;
  case ELF::EF_AMDGPU_MACH_AMDGCN_GFX1010: AK = GK_GFX1010; break;
  case ELF::EF_AMDGPU_MACH_AMDGCN_GFX1011: AK = GK_GFX1011; break;
  case ELF::EF_AMDGPU_MACH_AMDGCN_GFX1012: AK = GK_GFX1012; break;
  case ELF::EF_AMDGPU_MACH_AMDGCN_GFX1013: AK = GK_GFX1013; break;
  case ELF::EF_AMDGPU_MACH_AMDGCN_GFX1030: AK = GK_GFX1030; break;
  case ELF::EF_AMDGPU_MACH_AMDGCN_GFX1031: AK = GK_GFX1031; break;
  case ELF::EF_AMDGPU_MACH_AMDGCN_GFX1032: AK = GK_GFX1032; break;
  case ELF::EF_AMDGPU_MACH_AMDGCN_GFX1033: AK = GK_GFX1033; break;
  case ELF::EF_AMDGPU_MACH_AMDGCN_GFX1034: AK = GK_GFX1034; break;
  case ELF::EF_AMDGPU_MACH_AMDGCN_GFX1035: AK = GK_GFX1035; break;
  case ELF::EF_AMDGPU_MACH_AMDGCN_GFX1036: AK = GK_GFX1036; break;
  case ELF::EF_AMDGPU_MACH_AMDGCN_GFX1100: AK = GK_GFX1100; break;
  case ELF::EF_AMDGPU_MACH_AMDGCN_GFX1101: AK = GK_GFX1101; break;
  case ELF::EF_AMDGPU_MACH_AMDGCN_GFX1102: AK = GK_GFX1102; break;
  case ELF::EF_AMDGPU_MACH_AMDGCN_GFX1103: AK = GK_GFX1103; break;
  case ELF::EF_AMDGPU_MACH_AMDGCN_GFX1150: AK = GK_GFX1150; break;
  case ELF::EF_AMDGPU_MACH_AMDGCN_GFX1151: AK = GK_GFX1151; break;
  case ELF::EF_AMDGPU_MACH_AMDGCN_GFX1152: AK = GK_GFX1152; break;
  case ELF::EF_AMDGPU_MACH_AMDGCN_GFX1153: AK = GK_GFX1153; break;
  case ELF::EF_AMDGPU_MACH_AMDGCN_GFX1170: AK = GK_GFX1170; break;
  case ELF::EF_AMDGPU_MACH_AMDGCN_GFX1171: AK = GK_GFX1171; break;
  case ELF::EF_AMDGPU_MACH_AMDGCN_GFX1172: AK = GK_GFX1172; break;
  case ELF::EF_AMDGPU_MACH_AMDGCN_GFX1200: AK = GK_GFX1200; break;
  case ELF::EF_AMDGPU_MACH_AMDGCN_GFX1201: AK = GK_GFX1201; break;
  case ELF::EF_AMDGPU_MACH_AMDGCN_GFX1250: AK = GK_GFX1250; break;
  case ELF::EF_AMDGPU_MACH_AMDGCN_GFX1251: AK = GK_GFX1251; break;
  case ELF::EF_AMDGPU_MACH_AMDGCN_GFX1310: AK = GK_GFX1310; break;
  case ELF::EF_AMDGPU_MACH_AMDGCN_GFX9_GENERIC:     AK = GK_GFX9_GENERIC; break;
  case ELF::EF_AMDGPU_MACH_AMDGCN_GFX9_4_GENERIC:   AK = GK_GFX9_4_GENERIC; break;
  case ELF::EF_AMDGPU_MACH_AMDGCN_GFX10_1_GENERIC:  AK = GK_GFX10_1_GENERIC; break;
```
**EN:** This section advances the file implementation by combining declarations, control flow, and helper operations that support the AMDGPU backend.
**CN:** 本节通过组合声明、控制流与辅助操作来推进文件实现，支撑 AMDGPU 后端的整体功能。

### Lines 131-174: Declares getArchNameAMDGCN
```cpp
  case ELF::EF_AMDGPU_MACH_AMDGCN_GFX10_3_GENERIC:  AK = GK_GFX10_3_GENERIC; break;
  case ELF::EF_AMDGPU_MACH_AMDGCN_GFX11_GENERIC:    AK = GK_GFX11_GENERIC; break;
  case ELF::EF_AMDGPU_MACH_AMDGCN_GFX12_GENERIC:    AK = GK_GFX12_GENERIC; break;
  case ELF::EF_AMDGPU_MACH_AMDGCN_GFX12_5_GENERIC:  AK = GK_GFX12_5_GENERIC; break;
  case ELF::EF_AMDGPU_MACH_NONE:           AK = GK_NONE;    break;
  default:                                 AK = GK_NONE;    break;
  }
  // clang-format on

  StringRef GPUName = getArchNameAMDGCN(AK);
  if (GPUName != "")
    return GPUName;
  return getArchNameR600(AK);
}

unsigned AMDGPUTargetStreamer::getElfMach(StringRef GPU) {
  AMDGPU::GPUKind AK = parseArchAMDGCN(GPU);
  if (AK == AMDGPU::GPUKind::GK_NONE)
    AK = parseArchR600(GPU);

  // clang-format off
  switch (AK) {
  case GK_R600:    return ELF::EF_AMDGPU_MACH_R600_R600;
  case GK_R630:    return ELF::EF_AMDGPU_MACH_R600_R630;
  case GK_RS880:   return ELF::EF_AMDGPU_MACH_R600_RS880;
  case GK_RV670:   return ELF::EF_AMDGPU_MACH_R600_RV670;
  case GK_RV710:   return ELF::EF_AMDGPU_MACH_R600_RV710;
  case GK_RV730:   return ELF::EF_AMDGPU_MACH_R600_RV730;
  case GK_RV770:   return ELF::EF_AMDGPU_MACH_R600_RV770;
  case GK_CEDAR:   return ELF::EF_AMDGPU_MACH_R600_CEDAR;
  case GK_CYPRESS: return ELF::EF_AMDGPU_MACH_R600_CYPRESS;
  case GK_JUNIPER: return ELF::EF_AMDGPU_MACH_R600_JUNIPER;
  case GK_REDWOOD: return ELF::EF_AMDGPU_MACH_R600_REDWOOD;
  case GK_SUMO:    return ELF::EF_AMDGPU_MACH_R600_SUMO;
  case GK_BARTS:   return ELF::EF_AMDGPU_MACH_R600_BARTS;
  case GK_CAICOS:  return ELF::EF_AMDGPU_MACH_R600_CAICOS;
  case GK_CAYMAN:  return ELF::EF_AMDGPU_MACH_R600_CAYMAN;
  case GK_TURKS:   return ELF::EF_AMDGPU_MACH_R600_TURKS;
  case GK_GFX600:  return ELF::EF_AMDGPU_MACH_AMDGCN_GFX600;
  case GK_GFX601:  return ELF::EF_AMDGPU_MACH_AMDGCN_GFX601;
  case GK_GFX602:  return ELF::EF_AMDGPU_MACH_AMDGCN_GFX602;
  case GK_GFX700:  return ELF::EF_AMDGPU_MACH_AMDGCN_GFX700;
  case GK_GFX701:  return ELF::EF_AMDGPU_MACH_AMDGCN_GFX701;
  case GK_GFX702:  return ELF::EF_AMDGPU_MACH_AMDGCN_GFX702;
```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend. Main symbols: `AMDGPUTargetStreamer::getElfMach`.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。 主要符号：`AMDGPUTargetStreamer::getElfMach`。

### Lines 175-218: Result computation and returns
```cpp
  case GK_GFX703:  return ELF::EF_AMDGPU_MACH_AMDGCN_GFX703;
  case GK_GFX704:  return ELF::EF_AMDGPU_MACH_AMDGCN_GFX704;
  case GK_GFX705:  return ELF::EF_AMDGPU_MACH_AMDGCN_GFX705;
  case GK_GFX801:  return ELF::EF_AMDGPU_MACH_AMDGCN_GFX801;
  case GK_GFX802:  return ELF::EF_AMDGPU_MACH_AMDGCN_GFX802;
  case GK_GFX803:  return ELF::EF_AMDGPU_MACH_AMDGCN_GFX803;
  case GK_GFX805:  return ELF::EF_AMDGPU_MACH_AMDGCN_GFX805;
  case GK_GFX810:  return ELF::EF_AMDGPU_MACH_AMDGCN_GFX810;
  case GK_GFX900:  return ELF::EF_AMDGPU_MACH_AMDGCN_GFX900;
  case GK_GFX902:  return ELF::EF_AMDGPU_MACH_AMDGCN_GFX902;
  case GK_GFX904:  return ELF::EF_AMDGPU_MACH_AMDGCN_GFX904;
  case GK_GFX906:  return ELF::EF_AMDGPU_MACH_AMDGCN_GFX906;
  case GK_GFX908:  return ELF::EF_AMDGPU_MACH_AMDGCN_GFX908;
  case GK_GFX909:  return ELF::EF_AMDGPU_MACH_AMDGCN_GFX909;
  case GK_GFX90A:  return ELF::EF_AMDGPU_MACH_AMDGCN_GFX90A;
  case GK_GFX90C:  return ELF::EF_AMDGPU_MACH_AMDGCN_GFX90C;
  case GK_GFX942:  return ELF::EF_AMDGPU_MACH_AMDGCN_GFX942;
  case GK_GFX950:  return ELF::EF_AMDGPU_MACH_AMDGCN_GFX950;
  case GK_GFX1010: return ELF::EF_AMDGPU_MACH_AMDGCN_GFX1010;
  case GK_GFX1011: return ELF::EF_AMDGPU_MACH_AMDGCN_GFX1011;
  case GK_GFX1012: return ELF::EF_AMDGPU_MACH_AMDGCN_GFX1012;
  case GK_GFX1013: return ELF::EF_AMDGPU_MACH_AMDGCN_GFX1013;
  case GK_GFX1030: return ELF::EF_AMDGPU_MACH_AMDGCN_GFX1030;
  case GK_GFX1031: return ELF::EF_AMDGPU_MACH_AMDGCN_GFX1031;
  case GK_GFX1032: return ELF::EF_AMDGPU_MACH_AMDGCN_GFX1032;
  case GK_GFX1033: return ELF::EF_AMDGPU_MACH_AMDGCN_GFX1033;
  case GK_GFX1034: return ELF::EF_AMDGPU_MACH_AMDGCN_GFX1034;
  case GK_GFX1035: return ELF::EF_AMDGPU_MACH_AMDGCN_GFX1035;
  case GK_GFX1036: return ELF::EF_AMDGPU_MACH_AMDGCN_GFX1036;
  case GK_GFX1100: return ELF::EF_AMDGPU_MACH_AMDGCN_GFX1100;
  case GK_GFX1101: return ELF::EF_AMDGPU_MACH_AMDGCN_GFX1101;
  case GK_GFX1102: return ELF::EF_AMDGPU_MACH_AMDGCN_GFX1102;
  case GK_GFX1103: return ELF::EF_AMDGPU_MACH_AMDGCN_GFX1103;
  case GK_GFX1150: return ELF::EF_AMDGPU_MACH_AMDGCN_GFX1150;
  case GK_GFX1151: return ELF::EF_AMDGPU_MACH_AMDGCN_GFX1151;
  case GK_GFX1152: return ELF::EF_AMDGPU_MACH_AMDGCN_GFX1152;
  case GK_GFX1153: return ELF::EF_AMDGPU_MACH_AMDGCN_GFX1153;
  case GK_GFX1170: return ELF::EF_AMDGPU_MACH_AMDGCN_GFX1170;
  case GK_GFX1171: return ELF::EF_AMDGPU_MACH_AMDGCN_GFX1171;
  case GK_GFX1172: return ELF::EF_AMDGPU_MACH_AMDGCN_GFX1172;
  case GK_GFX1200: return ELF::EF_AMDGPU_MACH_AMDGCN_GFX1200;
  case GK_GFX1201: return ELF::EF_AMDGPU_MACH_AMDGCN_GFX1201;
  case GK_GFX1250: return ELF::EF_AMDGPU_MACH_AMDGCN_GFX1250;
  case GK_GFX1251: return ELF::EF_AMDGPU_MACH_AMDGCN_GFX1251;
```
**EN:** This section finalizes intermediate state and returns the value that the caller or pass pipeline needs.
**CN:** 本节整理中间状态并返回调用者或 Pass 流水线所需的结果。

### Lines 219-258: Result computation and returns
```cpp
  case GK_GFX1310: return ELF::EF_AMDGPU_MACH_AMDGCN_GFX1310;
  case GK_GFX9_GENERIC:     return ELF::EF_AMDGPU_MACH_AMDGCN_GFX9_GENERIC;
  case GK_GFX9_4_GENERIC:   return ELF::EF_AMDGPU_MACH_AMDGCN_GFX9_4_GENERIC;
  case GK_GFX10_1_GENERIC:  return ELF::EF_AMDGPU_MACH_AMDGCN_GFX10_1_GENERIC;
  case GK_GFX10_3_GENERIC:  return ELF::EF_AMDGPU_MACH_AMDGCN_GFX10_3_GENERIC;
  case GK_GFX11_GENERIC:    return ELF::EF_AMDGPU_MACH_AMDGCN_GFX11_GENERIC;
  case GK_GFX12_GENERIC:    return ELF::EF_AMDGPU_MACH_AMDGCN_GFX12_GENERIC;
  case GK_GFX12_5_GENERIC:  return ELF::EF_AMDGPU_MACH_AMDGCN_GFX12_5_GENERIC;
  case GK_NONE:    return ELF::EF_AMDGPU_MACH_NONE;
  }
  // clang-format on

  llvm_unreachable("unknown GPU");
}

//===----------------------------------------------------------------------===//
// AMDGPUTargetAsmStreamer
//===----------------------------------------------------------------------===//

AMDGPUTargetAsmStreamer::AMDGPUTargetAsmStreamer(MCStreamer &S,
                                                 formatted_raw_ostream &OS)
    : AMDGPUTargetStreamer(S), OS(OS) {}

// A hook for emitting stuff at the end.
// We use it for emitting the accumulated PAL metadata as directives.
// The PAL metadata is reset after it is emitted.
void AMDGPUTargetAsmStreamer::finish() {
  std::string S;
  getPALMetadata()->toString(S);
  OS << S;

  // Reset the pal metadata so its data will not affect a compilation that
  // reuses this object.
  getPALMetadata()->reset();
}

void AMDGPUTargetAsmStreamer::EmitDirectiveAMDGCNTarget() {
  OS << "\t.amdgcn_target \"" << *getTargetID() << "\"\n";
}

```
**EN:** This section finalizes intermediate state and returns the value that the caller or pass pipeline needs. Main symbols: `AMDGPUTargetAsmStreamer::AMDGPUTargetAsmStreamer`, `AMDGPUTargetAsmStreamer::finish`, `AMDGPUTargetAsmStreamer::EmitDirectiveAMDGCNTarget`.
**CN:** 本节整理中间状态并返回调用者或 Pass 流水线所需的结果。 主要符号：`AMDGPUTargetAsmStreamer::AMDGPUTargetAsmStreamer`, `AMDGPUTargetAsmStreamer::finish`, `AMDGPUTargetAsmStreamer::EmitDirectiveAMDGCNTarget`。

### Lines 259-292: Implements AMDGPUTargetAsmStreamer::EmitDirectiveAMDHSACodeObjectVersion
```cpp
void AMDGPUTargetAsmStreamer::EmitDirectiveAMDHSACodeObjectVersion(
    unsigned COV) {
  AMDGPUTargetStreamer::EmitDirectiveAMDHSACodeObjectVersion(COV);
  OS << "\t.amdhsa_code_object_version " << COV << '\n';
}

void AMDGPUTargetAsmStreamer::EmitAMDKernelCodeT(AMDGPUMCKernelCodeT &Header) {
  auto FoldAndPrint = [&](const MCExpr *Expr, raw_ostream &OS,
                          const MCAsmInfo *MAI) {
    printAMDGPUMCExpr(foldAMDGPUMCExpr(Expr, getContext()), OS, MAI);
  };

  OS << "\t.amd_kernel_code_t\n";
  Header.EmitKernelCodeT(OS, getContext(), FoldAndPrint);
  OS << "\t.end_amd_kernel_code_t\n";
}

void AMDGPUTargetAsmStreamer::EmitAMDGPUSymbolType(StringRef SymbolName,
                                                   unsigned Type) {
  switch (Type) {
  default:
    llvm_unreachable("Invalid AMDGPU symbol type");
  case ELF::STT_AMDGPU_HSA_KERNEL:
    OS << "\t.amdgpu_hsa_kernel " << SymbolName << '\n';
    break;
  }
}

void AMDGPUTargetAsmStreamer::emitAMDGPULDS(MCSymbol *Symbol, unsigned Size,
                                            Align Alignment) {
  OS << "\t.amdgpu_lds " << Symbol->getName() << ", " << Size << ", "
     << Alignment.value() << '\n';
}

```
**EN:** This section contains concrete logic for AMDGPUTargetAsmStreamer::EmitDirectiveAMDHSACodeObjectVersion. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPUTargetAsmStreamer::EmitDirectiveAMDHSACodeObjectVersion`, `AMDGPUTargetStreamer::EmitDirectiveAMDHSACodeObjectVersion`, `AMDGPUTargetAsmStreamer::EmitAMDKernelCodeT`.
**CN:** 本节包含与 AMDGPUTargetAsmStreamer::EmitDirectiveAMDHSACodeObjectVersion 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPUTargetAsmStreamer::EmitDirectiveAMDHSACodeObjectVersion`, `AMDGPUTargetStreamer::EmitDirectiveAMDHSACodeObjectVersion`, `AMDGPUTargetAsmStreamer::EmitAMDKernelCodeT`。

### Lines 293-335: Preprocessor guards and macros
```cpp
void AMDGPUTargetAsmStreamer::EmitMCResourceInfo(
    const MCSymbol *NumVGPR, const MCSymbol *NumAGPR,
    const MCSymbol *NumExplicitSGPR, const MCSymbol *NumNamedBarrier,
    const MCSymbol *PrivateSegmentSize, const MCSymbol *UsesVCC,
    const MCSymbol *UsesFlatScratch, const MCSymbol *HasDynamicallySizedStack,
    const MCSymbol *HasRecursion, const MCSymbol *HasIndirectCall) {
#define PRINT_RES_INFO(ARG)                                                    \
  OS << "\t.set ";                                                             \
  ARG->print(OS, &getContext().getAsmInfo());                                  \
  OS << ", ";                                                                  \
  getContext().getAsmInfo().printExpr(OS, *ARG->getVariableValue());           \
  Streamer.addBlankLine();

  PRINT_RES_INFO(NumVGPR);
  PRINT_RES_INFO(NumAGPR);
  PRINT_RES_INFO(NumExplicitSGPR);
  PRINT_RES_INFO(NumNamedBarrier);
  PRINT_RES_INFO(PrivateSegmentSize);
  PRINT_RES_INFO(UsesVCC);
  PRINT_RES_INFO(UsesFlatScratch);
  PRINT_RES_INFO(HasDynamicallySizedStack);
  PRINT_RES_INFO(HasRecursion);
  PRINT_RES_INFO(HasIndirectCall);
#undef PRINT_RES_INFO
}

void AMDGPUTargetAsmStreamer::EmitMCResourceMaximums(
    const MCSymbol *MaxVGPR, const MCSymbol *MaxAGPR, const MCSymbol *MaxSGPR,
    const MCSymbol *MaxNamedBarrier) {
#define PRINT_RES_INFO(ARG)                                                    \
  OS << "\t.set ";                                                             \
  ARG->print(OS, &getContext().getAsmInfo());                                  \
  OS << ", ";                                                                  \
  getContext().getAsmInfo().printExpr(OS, *ARG->getVariableValue());           \
  Streamer.addBlankLine();

  PRINT_RES_INFO(MaxVGPR);
  PRINT_RES_INFO(MaxAGPR);
  PRINT_RES_INFO(MaxSGPR);
  PRINT_RES_INFO(MaxNamedBarrier);
#undef PRINT_RES_INFO
}

```
**EN:** These lines define compile-time structure such as include guards, feature switches, or macros that shape how the file is compiled. Main symbols: `AMDGPUTargetAsmStreamer::EmitMCResourceInfo`, `AMDGPUTargetAsmStreamer::EmitMCResourceMaximums`.
**CN:** 这些语句定义了编译期结构，例如 include guard、特性开关或宏，用于决定文件的编译方式。 主要符号：`AMDGPUTargetAsmStreamer::EmitMCResourceInfo`, `AMDGPUTargetAsmStreamer::EmitMCResourceMaximums`。

### Lines 336-378: Implements AMDGPUTargetAsmStreamer::EmitISAVersion
```cpp
bool AMDGPUTargetAsmStreamer::EmitISAVersion() {
  OS << "\t.amd_amdgpu_isa \"" << getTargetID() << "\"\n";
  return true;
}

bool AMDGPUTargetAsmStreamer::EmitHSAMetadata(msgpack::Document &HSAMetadataDoc,
                                              bool Strict) {
  HSAMD::V3::MetadataVerifier Verifier(Strict);
  if (!Verifier.verify(HSAMetadataDoc.getRoot()))
    return false;

  std::string HSAMetadataString;
  raw_string_ostream StrOS(HSAMetadataString);
  HSAMetadataDoc.toYAML(StrOS);

  OS << '\t' << HSAMD::V3::AssemblerDirectiveBegin << '\n';
  OS << StrOS.str() << '\n';
  OS << '\t' << HSAMD::V3::AssemblerDirectiveEnd << '\n';
  return true;
}

bool AMDGPUTargetAsmStreamer::EmitCodeEnd(const MCSubtargetInfo &STI) {
  const uint32_t Encoded_s_code_end = 0xbf9f0000;
  const uint32_t Encoded_s_nop = 0xbf800000;
  uint32_t Encoded_pad = Encoded_s_code_end;

  // Instruction cache line size in bytes.
  const unsigned Log2CacheLineSize = AMDGPU::isGFX11Plus(STI) ? 7 : 6;
  const unsigned CacheLineSize = 1u << Log2CacheLineSize;

  // Extra padding amount in bytes to support prefetch mode 3.
  unsigned FillSize = 3 * CacheLineSize;

  if (AMDGPU::isGFX90A(STI)) {
    Encoded_pad = Encoded_s_nop;
    FillSize = 16 * CacheLineSize;
  }

  OS << "\t.p2alignl " << Log2CacheLineSize << ", " << Encoded_pad << '\n';
  OS << "\t.fill " << (FillSize / 4) << ", 4, " << Encoded_pad << '\n';
  return true;
}

```
**EN:** This section contains concrete logic for AMDGPUTargetAsmStreamer::EmitISAVersion. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPUTargetAsmStreamer::EmitISAVersion`, `AMDGPUTargetAsmStreamer::EmitHSAMetadata`, `AMDGPUTargetAsmStreamer::EmitCodeEnd`.
**CN:** 本节包含与 AMDGPUTargetAsmStreamer::EmitISAVersion 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPUTargetAsmStreamer::EmitISAVersion`, `AMDGPUTargetAsmStreamer::EmitHSAMetadata`, `AMDGPUTargetAsmStreamer::EmitCodeEnd`。

### Lines 379-415: Implements AMDGPUTargetAsmStreamer::EmitAmdhsaKernelDescriptor
```cpp
void AMDGPUTargetAsmStreamer::EmitAmdhsaKernelDescriptor(
    const MCSubtargetInfo &STI, StringRef KernelName,
    const MCKernelDescriptor &KD, const MCExpr *NextVGPR,
    const MCExpr *NextSGPR, const MCExpr *ReserveVCC,
    const MCExpr *ReserveFlatScr) {
  IsaVersion IVersion = getIsaVersion(STI.getCPU());
  const MCAsmInfo &MAI = getContext().getAsmInfo();

  OS << "\t.amdhsa_kernel " << KernelName << '\n';

  auto PrintField = [&](const MCExpr *Expr, uint32_t Shift, uint32_t Mask,
                        StringRef Directive) {
    OS << "\t\t" << Directive << ' ';
    const MCExpr *ShiftedAndMaskedExpr =
        MCKernelDescriptor::bits_get(Expr, Shift, Mask, getContext());
    const MCExpr *New = foldAMDGPUMCExpr(ShiftedAndMaskedExpr, getContext());
    printAMDGPUMCExpr(New, OS, &MAI);
    OS << '\n';
  };

  auto EmitMCExpr = [&](const MCExpr *Value) {
    const MCExpr *NewExpr = foldAMDGPUMCExpr(Value, getContext());
    printAMDGPUMCExpr(NewExpr, OS, &MAI);
  };

  OS << "\t\t.amdhsa_group_segment_fixed_size ";
  EmitMCExpr(KD.group_segment_fixed_size);
  OS << '\n';

  OS << "\t\t.amdhsa_private_segment_fixed_size ";
  EmitMCExpr(KD.private_segment_fixed_size);
  OS << '\n';

  OS << "\t\t.amdhsa_kernarg_size ";
  EmitMCExpr(KD.kernarg_size);
  OS << '\n';

```
**EN:** This section contains concrete logic for AMDGPUTargetAsmStreamer::EmitAmdhsaKernelDescriptor. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPUTargetAsmStreamer::EmitAmdhsaKernelDescriptor`, `MCKernelDescriptor::bits_get`.
**CN:** 本节包含与 AMDGPUTargetAsmStreamer::EmitAmdhsaKernelDescriptor 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPUTargetAsmStreamer::EmitAmdhsaKernelDescriptor`, `MCKernelDescriptor::bits_get`。

### Lines 416-459: Conditional logic and checks
```cpp
  if (isGFX1250Plus(STI)) {
    PrintField(KD.compute_pgm_rsrc2,
               amdhsa::COMPUTE_PGM_RSRC2_GFX125_USER_SGPR_COUNT_SHIFT,
               amdhsa::COMPUTE_PGM_RSRC2_GFX125_USER_SGPR_COUNT,
               ".amdhsa_user_sgpr_count");
  } else {
    PrintField(KD.compute_pgm_rsrc2,
               amdhsa::COMPUTE_PGM_RSRC2_GFX6_GFX120_USER_SGPR_COUNT_SHIFT,
               amdhsa::COMPUTE_PGM_RSRC2_GFX6_GFX120_USER_SGPR_COUNT,
               ".amdhsa_user_sgpr_count");
  }

  if (!hasArchitectedFlatScratch(STI))
    PrintField(
        KD.kernel_code_properties,
        amdhsa::KERNEL_CODE_PROPERTY_ENABLE_SGPR_PRIVATE_SEGMENT_BUFFER_SHIFT,
        amdhsa::KERNEL_CODE_PROPERTY_ENABLE_SGPR_PRIVATE_SEGMENT_BUFFER,
        ".amdhsa_user_sgpr_private_segment_buffer");
  PrintField(KD.kernel_code_properties,
             amdhsa::KERNEL_CODE_PROPERTY_ENABLE_SGPR_DISPATCH_PTR_SHIFT,
             amdhsa::KERNEL_CODE_PROPERTY_ENABLE_SGPR_DISPATCH_PTR,
             ".amdhsa_user_sgpr_dispatch_ptr");
  PrintField(KD.kernel_code_properties,
             amdhsa::KERNEL_CODE_PROPERTY_ENABLE_SGPR_QUEUE_PTR_SHIFT,
             amdhsa::KERNEL_CODE_PROPERTY_ENABLE_SGPR_QUEUE_PTR,
             ".amdhsa_user_sgpr_queue_ptr");
  PrintField(KD.kernel_code_properties,
             amdhsa::KERNEL_CODE_PROPERTY_ENABLE_SGPR_KERNARG_SEGMENT_PTR_SHIFT,
             amdhsa::KERNEL_CODE_PROPERTY_ENABLE_SGPR_KERNARG_SEGMENT_PTR,
             ".amdhsa_user_sgpr_kernarg_segment_ptr");
  PrintField(KD.kernel_code_properties,
             amdhsa::KERNEL_CODE_PROPERTY_ENABLE_SGPR_DISPATCH_ID_SHIFT,
             amdhsa::KERNEL_CODE_PROPERTY_ENABLE_SGPR_DISPATCH_ID,
             ".amdhsa_user_sgpr_dispatch_id");
  if (!hasArchitectedFlatScratch(STI))
    PrintField(KD.kernel_code_properties,
               amdhsa::KERNEL_CODE_PROPERTY_ENABLE_SGPR_FLAT_SCRATCH_INIT_SHIFT,
               amdhsa::KERNEL_CODE_PROPERTY_ENABLE_SGPR_FLAT_SCRATCH_INIT,
               ".amdhsa_user_sgpr_flat_scratch_init");
  if (hasKernargPreload(STI)) {
    PrintField(KD.kernarg_preload, amdhsa::KERNARG_PRELOAD_SPEC_LENGTH_SHIFT,
               amdhsa::KERNARG_PRELOAD_SPEC_LENGTH,
               ".amdhsa_user_sgpr_kernarg_preload_length");
    PrintField(KD.kernarg_preload, amdhsa::KERNARG_PRELOAD_SPEC_OFFSET_SHIFT,
```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 460-503: Defines PrintField
```cpp
               amdhsa::KERNARG_PRELOAD_SPEC_OFFSET,
               ".amdhsa_user_sgpr_kernarg_preload_offset");
  }
  PrintField(
      KD.kernel_code_properties,
      amdhsa::KERNEL_CODE_PROPERTY_ENABLE_SGPR_PRIVATE_SEGMENT_SIZE_SHIFT,
      amdhsa::KERNEL_CODE_PROPERTY_ENABLE_SGPR_PRIVATE_SEGMENT_SIZE,
      ".amdhsa_user_sgpr_private_segment_size");
  if (IVersion.Major >= 10)
    PrintField(KD.kernel_code_properties,
               amdhsa::KERNEL_CODE_PROPERTY_ENABLE_WAVEFRONT_SIZE32_SHIFT,
               amdhsa::KERNEL_CODE_PROPERTY_ENABLE_WAVEFRONT_SIZE32,
               ".amdhsa_wavefront_size32");
  if (CodeObjectVersion >= AMDGPU::AMDHSA_COV5)
    PrintField(KD.kernel_code_properties,
               amdhsa::KERNEL_CODE_PROPERTY_USES_DYNAMIC_STACK_SHIFT,
               amdhsa::KERNEL_CODE_PROPERTY_USES_DYNAMIC_STACK,
               ".amdhsa_uses_dynamic_stack");
  PrintField(KD.compute_pgm_rsrc2,
             amdhsa::COMPUTE_PGM_RSRC2_ENABLE_PRIVATE_SEGMENT_SHIFT,
             amdhsa::COMPUTE_PGM_RSRC2_ENABLE_PRIVATE_SEGMENT,
             (hasArchitectedFlatScratch(STI)
                  ? ".amdhsa_enable_private_segment"
                  : ".amdhsa_system_sgpr_private_segment_wavefront_offset"));
  PrintField(KD.compute_pgm_rsrc2,
             amdhsa::COMPUTE_PGM_RSRC2_ENABLE_SGPR_WORKGROUP_ID_X_SHIFT,
             amdhsa::COMPUTE_PGM_RSRC2_ENABLE_SGPR_WORKGROUP_ID_X,
             ".amdhsa_system_sgpr_workgroup_id_x");
  PrintField(KD.compute_pgm_rsrc2,
             amdhsa::COMPUTE_PGM_RSRC2_ENABLE_SGPR_WORKGROUP_ID_Y_SHIFT,
             amdhsa::COMPUTE_PGM_RSRC2_ENABLE_SGPR_WORKGROUP_ID_Y,
             ".amdhsa_system_sgpr_workgroup_id_y");
  PrintField(KD.compute_pgm_rsrc2,
             amdhsa::COMPUTE_PGM_RSRC2_ENABLE_SGPR_WORKGROUP_ID_Z_SHIFT,
             amdhsa::COMPUTE_PGM_RSRC2_ENABLE_SGPR_WORKGROUP_ID_Z,
             ".amdhsa_system_sgpr_workgroup_id_z");
  PrintField(KD.compute_pgm_rsrc2,
             amdhsa::COMPUTE_PGM_RSRC2_ENABLE_SGPR_WORKGROUP_INFO_SHIFT,
             amdhsa::COMPUTE_PGM_RSRC2_ENABLE_SGPR_WORKGROUP_INFO,
             ".amdhsa_system_sgpr_workgroup_info");
  PrintField(KD.compute_pgm_rsrc2,
             amdhsa::COMPUTE_PGM_RSRC2_ENABLE_VGPR_WORKITEM_ID_SHIFT,
             amdhsa::COMPUTE_PGM_RSRC2_ENABLE_VGPR_WORKITEM_ID,
             ".amdhsa_system_vgpr_workitem_id");
```
**EN:** This section contains concrete logic for PrintField. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 PrintField 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 504-545: Declares EmitMCExpr
```cpp

  // These directives are required.
  OS << "\t\t.amdhsa_next_free_vgpr ";
  EmitMCExpr(NextVGPR);
  OS << '\n';

  OS << "\t\t.amdhsa_next_free_sgpr ";
  EmitMCExpr(NextSGPR);
  OS << '\n';

  if (AMDGPU::isGFX90A(STI)) {
    // MCExpr equivalent of taking the (accum_offset + 1) * 4.
    const MCExpr *accum_bits = MCKernelDescriptor::bits_get(
        KD.compute_pgm_rsrc3,
        amdhsa::COMPUTE_PGM_RSRC3_GFX90A_ACCUM_OFFSET_SHIFT,
        amdhsa::COMPUTE_PGM_RSRC3_GFX90A_ACCUM_OFFSET, getContext());
    accum_bits = MCBinaryExpr::createAdd(
        accum_bits, MCConstantExpr::create(1, getContext()), getContext());
    accum_bits = MCBinaryExpr::createMul(
        accum_bits, MCConstantExpr::create(4, getContext()), getContext());
    OS << "\t\t.amdhsa_accum_offset ";
    const MCExpr *New = foldAMDGPUMCExpr(accum_bits, getContext());
    printAMDGPUMCExpr(New, OS, &MAI);
    OS << '\n';
  }

  if (isGFX1250Plus(STI))
    PrintField(KD.compute_pgm_rsrc3,
               amdhsa::COMPUTE_PGM_RSRC3_GFX125_NAMED_BAR_CNT_SHIFT,
               amdhsa::COMPUTE_PGM_RSRC3_GFX125_NAMED_BAR_CNT,
               ".amdhsa_named_barrier_count");

  OS << "\t\t.amdhsa_reserve_vcc ";
  EmitMCExpr(ReserveVCC);
  OS << '\n';

  if (IVersion.Major >= 7 && !hasArchitectedFlatScratch(STI)) {
    OS << "\t\t.amdhsa_reserve_flat_scratch ";
    EmitMCExpr(ReserveFlatScr);
    OS << '\n';
  }

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend. Main symbols: `AMDGPU::isGFX90A`, `MCKernelDescriptor::bits_get`, `MCBinaryExpr::createAdd`.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。 主要符号：`AMDGPU::isGFX90A`, `MCKernelDescriptor::bits_get`, `MCBinaryExpr::createAdd`。

### Lines 546-589: Defines PrintField
```cpp
  switch (CodeObjectVersion) {
  default:
    break;
  case AMDGPU::AMDHSA_COV4:
  case AMDGPU::AMDHSA_COV5:
    if (getTargetID()->isXnackSupported())
      OS << "\t\t.amdhsa_reserve_xnack_mask " << getTargetID()->isXnackOnOrAny()
         << '\n';
    break;
  }

  PrintField(KD.compute_pgm_rsrc1,
             amdhsa::COMPUTE_PGM_RSRC1_FLOAT_ROUND_MODE_32_SHIFT,
             amdhsa::COMPUTE_PGM_RSRC1_FLOAT_ROUND_MODE_32,
             ".amdhsa_float_round_mode_32");
  PrintField(KD.compute_pgm_rsrc1,
             amdhsa::COMPUTE_PGM_RSRC1_FLOAT_ROUND_MODE_16_64_SHIFT,
             amdhsa::COMPUTE_PGM_RSRC1_FLOAT_ROUND_MODE_16_64,
             ".amdhsa_float_round_mode_16_64");
  PrintField(KD.compute_pgm_rsrc1,
             amdhsa::COMPUTE_PGM_RSRC1_FLOAT_DENORM_MODE_32_SHIFT,
             amdhsa::COMPUTE_PGM_RSRC1_FLOAT_DENORM_MODE_32,
             ".amdhsa_float_denorm_mode_32");
  PrintField(KD.compute_pgm_rsrc1,
             amdhsa::COMPUTE_PGM_RSRC1_FLOAT_DENORM_MODE_16_64_SHIFT,
             amdhsa::COMPUTE_PGM_RSRC1_FLOAT_DENORM_MODE_16_64,
             ".amdhsa_float_denorm_mode_16_64");
  if (STI.hasFeature(AMDGPU::FeatureDX10ClampAndIEEEMode)) {
    PrintField(KD.compute_pgm_rsrc1,
               amdhsa::COMPUTE_PGM_RSRC1_GFX6_GFX11_ENABLE_DX10_CLAMP_SHIFT,
               amdhsa::COMPUTE_PGM_RSRC1_GFX6_GFX11_ENABLE_DX10_CLAMP,
               ".amdhsa_dx10_clamp");
    PrintField(KD.compute_pgm_rsrc1,
               amdhsa::COMPUTE_PGM_RSRC1_GFX6_GFX11_ENABLE_IEEE_MODE_SHIFT,
               amdhsa::COMPUTE_PGM_RSRC1_GFX6_GFX11_ENABLE_IEEE_MODE,
               ".amdhsa_ieee_mode");
  }
  if (IVersion.Major >= 9) {
    PrintField(KD.compute_pgm_rsrc1,
               amdhsa::COMPUTE_PGM_RSRC1_GFX9_PLUS_FP16_OVFL_SHIFT,
               amdhsa::COMPUTE_PGM_RSRC1_GFX9_PLUS_FP16_OVFL,
               ".amdhsa_fp16_overflow");
  }
  if (AMDGPU::isGFX90A(STI))
```
**EN:** This section contains concrete logic for PrintField. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPU::isGFX90A`.
**CN:** 本节包含与 PrintField 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPU::isGFX90A`。

### Lines 590-633: Conditional logic and checks
```cpp
    PrintField(KD.compute_pgm_rsrc3,
               amdhsa::COMPUTE_PGM_RSRC3_GFX90A_TG_SPLIT_SHIFT,
               amdhsa::COMPUTE_PGM_RSRC3_GFX90A_TG_SPLIT, ".amdhsa_tg_split");
  if (AMDGPU::supportsWGP(STI))
    PrintField(KD.compute_pgm_rsrc1,
               amdhsa::COMPUTE_PGM_RSRC1_GFX10_PLUS_WGP_MODE_SHIFT,
               amdhsa::COMPUTE_PGM_RSRC1_GFX10_PLUS_WGP_MODE,
               ".amdhsa_workgroup_processor_mode");
  if (IVersion.Major >= 10) {
    PrintField(KD.compute_pgm_rsrc1,
               amdhsa::COMPUTE_PGM_RSRC1_GFX10_PLUS_MEM_ORDERED_SHIFT,
               amdhsa::COMPUTE_PGM_RSRC1_GFX10_PLUS_MEM_ORDERED,
               ".amdhsa_memory_ordered");
    PrintField(KD.compute_pgm_rsrc1,
               amdhsa::COMPUTE_PGM_RSRC1_GFX10_PLUS_FWD_PROGRESS_SHIFT,
               amdhsa::COMPUTE_PGM_RSRC1_GFX10_PLUS_FWD_PROGRESS,
               ".amdhsa_forward_progress");
  }
  if (IVersion.Major >= 10 && IVersion.Major < 12) {
    PrintField(KD.compute_pgm_rsrc3,
               amdhsa::COMPUTE_PGM_RSRC3_GFX10_GFX11_SHARED_VGPR_COUNT_SHIFT,
               amdhsa::COMPUTE_PGM_RSRC3_GFX10_GFX11_SHARED_VGPR_COUNT,
               ".amdhsa_shared_vgpr_count");
  }
  if (IVersion.Major == 11) {
    PrintField(KD.compute_pgm_rsrc3,
               amdhsa::COMPUTE_PGM_RSRC3_GFX11_INST_PREF_SIZE_SHIFT,
               amdhsa::COMPUTE_PGM_RSRC3_GFX11_INST_PREF_SIZE,
               ".amdhsa_inst_pref_size");
  }
  if (IVersion.Major >= 12) {
    PrintField(KD.compute_pgm_rsrc3,
               amdhsa::COMPUTE_PGM_RSRC3_GFX12_PLUS_INST_PREF_SIZE_SHIFT,
               amdhsa::COMPUTE_PGM_RSRC3_GFX12_PLUS_INST_PREF_SIZE,
               ".amdhsa_inst_pref_size");
    PrintField(KD.compute_pgm_rsrc1,
               amdhsa::COMPUTE_PGM_RSRC1_GFX12_PLUS_ENABLE_WG_RR_EN_SHIFT,
               amdhsa::COMPUTE_PGM_RSRC1_GFX12_PLUS_ENABLE_WG_RR_EN,
               ".amdhsa_round_robin_scheduling");
  }
  PrintField(
      KD.compute_pgm_rsrc2,
      amdhsa::
          COMPUTE_PGM_RSRC2_ENABLE_EXCEPTION_IEEE_754_FP_INVALID_OPERATION_SHIFT,
```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `AMDGPU::supportsWGP`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`AMDGPU::supportsWGP`。

### Lines 634-670: Defines PrintField
```cpp
      amdhsa::COMPUTE_PGM_RSRC2_ENABLE_EXCEPTION_IEEE_754_FP_INVALID_OPERATION,
      ".amdhsa_exception_fp_ieee_invalid_op");
  PrintField(
      KD.compute_pgm_rsrc2,
      amdhsa::COMPUTE_PGM_RSRC2_ENABLE_EXCEPTION_FP_DENORMAL_SOURCE_SHIFT,
      amdhsa::COMPUTE_PGM_RSRC2_ENABLE_EXCEPTION_FP_DENORMAL_SOURCE,
      ".amdhsa_exception_fp_denorm_src");
  PrintField(
      KD.compute_pgm_rsrc2,
      amdhsa::
          COMPUTE_PGM_RSRC2_ENABLE_EXCEPTION_IEEE_754_FP_DIVISION_BY_ZERO_SHIFT,
      amdhsa::COMPUTE_PGM_RSRC2_ENABLE_EXCEPTION_IEEE_754_FP_DIVISION_BY_ZERO,
      ".amdhsa_exception_fp_ieee_div_zero");
  PrintField(
      KD.compute_pgm_rsrc2,
      amdhsa::COMPUTE_PGM_RSRC2_ENABLE_EXCEPTION_IEEE_754_FP_OVERFLOW_SHIFT,
      amdhsa::COMPUTE_PGM_RSRC2_ENABLE_EXCEPTION_IEEE_754_FP_OVERFLOW,
      ".amdhsa_exception_fp_ieee_overflow");
  PrintField(
      KD.compute_pgm_rsrc2,
      amdhsa::COMPUTE_PGM_RSRC2_ENABLE_EXCEPTION_IEEE_754_FP_UNDERFLOW_SHIFT,
      amdhsa::COMPUTE_PGM_RSRC2_ENABLE_EXCEPTION_IEEE_754_FP_UNDERFLOW,
      ".amdhsa_exception_fp_ieee_underflow");
  PrintField(
      KD.compute_pgm_rsrc2,
      amdhsa::COMPUTE_PGM_RSRC2_ENABLE_EXCEPTION_IEEE_754_FP_INEXACT_SHIFT,
      amdhsa::COMPUTE_PGM_RSRC2_ENABLE_EXCEPTION_IEEE_754_FP_INEXACT,
      ".amdhsa_exception_fp_ieee_inexact");
  PrintField(
      KD.compute_pgm_rsrc2,
      amdhsa::COMPUTE_PGM_RSRC2_ENABLE_EXCEPTION_INT_DIVIDE_BY_ZERO_SHIFT,
      amdhsa::COMPUTE_PGM_RSRC2_ENABLE_EXCEPTION_INT_DIVIDE_BY_ZERO,
      ".amdhsa_exception_int_div_zero");

  OS << "\t.end_amdhsa_kernel\n";
}

```
**EN:** This section contains concrete logic for PrintField. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 PrintField 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 671-699: Defines void
```cpp
namespace {
/// Callback type invoked by \c forEachInfoScope for each function scope in
/// the canonical iteration order. The scope is emitted exactly once per
/// unique \p Sym regardless of how many flat entries reference it.
using InfoScopeEmitter = function_ref<void(
    MCSymbol *Sym, const AMDGPU::FuncInfo *Info, ArrayRef<MCSymbol *> Uses,
    ArrayRef<MCSymbol *> Calls, ArrayRef<StringRef> IndirectCallTypeIds,
    ArrayRef<StringRef> TypeIds)>;

/// Group the flat edge lists in \p Data by source function symbol and drive
/// per-scope emission. A scope is opened for every function with attached
/// info and for every function that appears only as an edge source; each
/// scope is emitted exactly once. Both the asm and ELF streamers share this
/// iteration logic and only differ in the per-scope emission callback.
static void forEachInfoScope(const AMDGPU::InfoSectionData &Data,
                             InfoScopeEmitter Emit) {
  DenseMap<MCSymbol *, SmallVector<MCSymbol *, 2>> FuncUses;
  DenseMap<MCSymbol *, SmallVector<MCSymbol *, 4>> FuncCalls;
  DenseMap<MCSymbol *, SmallVector<StringRef, 2>> FuncIndirectCalls;
  DenseMap<MCSymbol *, SmallVector<StringRef, 1>> FuncTypeIds;
  for (const auto &[Func, Res] : Data.Uses)
    FuncUses[Func].push_back(Res);
  for (const auto &[Src, Dst] : Data.Calls)
    FuncCalls[Src].push_back(Dst);
  for (const auto &[Func, TypeId] : Data.IndirectCalls)
    FuncIndirectCalls[Func].push_back(TypeId);
  for (const auto &[Sym, TypeId] : Data.TypeIds)
    FuncTypeIds[Sym].push_back(TypeId);

```
**EN:** This section contains concrete logic for void. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 void 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 700-731: Conditional logic and checks
```cpp
  DenseSet<MCSymbol *> Emitted;
  auto EmitIfNew = [&](MCSymbol *Sym, const AMDGPU::FuncInfo *Info) {
    if (!Emitted.insert(Sym).second)
      return;
    ArrayRef<MCSymbol *> Uses, Calls;
    ArrayRef<StringRef> IndirectCallTypeIds, TypeIds;
    if (auto It = FuncUses.find(Sym); It != FuncUses.end())
      Uses = It->second;
    if (auto It = FuncCalls.find(Sym); It != FuncCalls.end())
      Calls = It->second;
    if (auto It = FuncIndirectCalls.find(Sym); It != FuncIndirectCalls.end())
      IndirectCallTypeIds = It->second;
    if (auto It = FuncTypeIds.find(Sym); It != FuncTypeIds.end())
      TypeIds = It->second;
    Emit(Sym, Info, Uses, Calls, IndirectCallTypeIds, TypeIds);
  };

  for (const AMDGPU::FuncInfo &Func : Data.Funcs)
    EmitIfNew(Func.Sym, &Func);
  // Emit scopes for functions that only appear as edge sources (e.g. typeid
  // tags on address-taken declarations, or callers of external functions).
  for (const auto &[Sym, TypeId] : Data.TypeIds)
    EmitIfNew(Sym, nullptr);
  for (const auto &[Sym, Res] : Data.Uses)
    EmitIfNew(Sym, nullptr);
  for (const auto &[Sym, Dst] : Data.Calls)
    EmitIfNew(Sym, nullptr);
  for (const auto &[Sym, TypeId] : Data.IndirectCalls)
    EmitIfNew(Sym, nullptr);
}
} // namespace

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 732-775: Implements AMDGPUTargetAsmStreamer::emitAMDGPUInfo
```cpp
void AMDGPUTargetAsmStreamer::emitAMDGPUInfo(
    const AMDGPU::InfoSectionData &Data) {
  forEachInfoScope(Data, [&](MCSymbol *Sym, const AMDGPU::FuncInfo *Info,
                             ArrayRef<MCSymbol *> Uses,
                             ArrayRef<MCSymbol *> Calls,
                             ArrayRef<StringRef> IndirectCallTypeIds,
                             ArrayRef<StringRef> TypeIds) {
    OS << "\t.amdgpu_info " << Sym->getName() << '\n';
    if (Info) {
      AMDGPU::FuncInfoFlags Flags{};
      if (Info->UsesVCC)
        Flags |= AMDGPU::FuncInfoFlags::FUNC_USES_VCC;
      if (Info->UsesFlatScratch)
        Flags |= AMDGPU::FuncInfoFlags::FUNC_USES_FLAT_SCRATCH;
      if (Info->HasDynStack)
        Flags |= AMDGPU::FuncInfoFlags::FUNC_HAS_DYN_STACK;
      OS << "\t\t.amdgpu_flags " << llvm::to_underlying(Flags) << '\n';
      OS << "\t\t.amdgpu_num_sgpr " << Info->NumSGPR << '\n';
      OS << "\t\t.amdgpu_num_vgpr " << Info->NumArchVGPR << '\n';
      if (Info->NumAccVGPR)
        OS << "\t\t.amdgpu_num_agpr " << Info->NumAccVGPR << '\n';
      OS << "\t\t.amdgpu_private_segment_size " << Info->PrivateSegmentSize
         << '\n';
    }
    for (MCSymbol *Res : Uses)
      OS << "\t\t.amdgpu_use " << Res->getName() << '\n';
    for (MCSymbol *Dst : Calls)
      OS << "\t\t.amdgpu_call " << Dst->getName() << '\n';
    for (StringRef TypeId : IndirectCallTypeIds)
      OS << "\t\t.amdgpu_indirect_call \"" << TypeId << "\"\n";
    for (StringRef TypeId : TypeIds)
      OS << "\t\t.amdgpu_typeid \"" << TypeId << "\"\n";
    OS << "\t.end_amdgpu_info\n\n";
  });
}

//===----------------------------------------------------------------------===//
// AMDGPUTargetELFStreamer
//===----------------------------------------------------------------------===//

AMDGPUTargetELFStreamer::AMDGPUTargetELFStreamer(MCStreamer &S,
                                                 const MCSubtargetInfo &STI)
    : AMDGPUTargetStreamer(S), STI(STI), Streamer(S) {}

```
**EN:** This section contains concrete logic for AMDGPUTargetAsmStreamer::emitAMDGPUInfo. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPUTargetAsmStreamer::emitAMDGPUInfo`, `llvm::to_underlying`, `AMDGPUTargetELFStreamer::AMDGPUTargetELFStreamer`.
**CN:** 本节包含与 AMDGPUTargetAsmStreamer::emitAMDGPUInfo 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPUTargetAsmStreamer::emitAMDGPUInfo`, `llvm::to_underlying`, `AMDGPUTargetELFStreamer::AMDGPUTargetELFStreamer`。

### Lines 776-816: Implements AMDGPUTargetELFStreamer::getStreamer
```cpp
MCELFStreamer &AMDGPUTargetELFStreamer::getStreamer() {
  return static_cast<MCELFStreamer &>(Streamer);
}

// A hook for emitting stuff at the end.
// We use it for emitting the accumulated PAL metadata as a .note record.
// The PAL metadata is reset after it is emitted.
void AMDGPUTargetELFStreamer::finish() {
  ELFObjectWriter &W = getStreamer().getWriter();
  W.setELFHeaderEFlags(getEFlags());
  W.setOverrideABIVersion(
      getELFABIVersion(STI.getTargetTriple(), CodeObjectVersion));

  std::string Blob;
  const char *Vendor = getPALMetadata()->getVendor();
  unsigned Type = getPALMetadata()->getType();
  getPALMetadata()->toBlob(Type, Blob);
  if (Blob.empty())
    return;
  EmitNote(Vendor, MCConstantExpr::create(Blob.size(), getContext()), Type,
           [&](MCELFStreamer &OS) { OS.emitBytes(Blob); });

  // Reset the pal metadata so its data will not affect a compilation that
  // reuses this object.
  getPALMetadata()->reset();
}

void AMDGPUTargetELFStreamer::EmitNote(
    StringRef Name, const MCExpr *DescSZ, unsigned NoteType,
    function_ref<void(MCELFStreamer &)> EmitDesc) {
  auto &S = getStreamer();
  auto &Context = S.getContext();

  auto NameSZ = Name.size() + 1;

  unsigned NoteFlags = 0;
  // TODO Apparently, this is currently needed for OpenCL as mentioned in
  // https://reviews.llvm.org/D74995
  if (isHsaAbi(STI))
    NoteFlags = ELF::SHF_ALLOC;

```
**EN:** This section contains concrete logic for AMDGPUTargetELFStreamer::getStreamer. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPUTargetELFStreamer::getStreamer`, `AMDGPUTargetELFStreamer::finish`, `MCConstantExpr::create`.
**CN:** 本节包含与 AMDGPUTargetELFStreamer::getStreamer 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPUTargetELFStreamer::getStreamer`, `AMDGPUTargetELFStreamer::finish`, `MCConstantExpr::create`。

### Lines 817-849: Declares pushSection
```cpp
  S.pushSection();
  S.switchSection(
      Context.getELFSection(ElfNote::SectionName, ELF::SHT_NOTE, NoteFlags));
  S.emitInt32(NameSZ);                       // namesz
  S.emitValue(DescSZ, 4);                    // descz
  S.emitInt32(NoteType);                     // type
  S.emitBytes(Name);                         // name
  S.emitValueToAlignment(Align(4), 0, 1, 0); // padding 0
  EmitDesc(S);                               // desc
  S.emitValueToAlignment(Align(4), 0, 1, 0); // padding 0
  S.popSection();
}

unsigned AMDGPUTargetELFStreamer::getEFlags() {
  switch (STI.getTargetTriple().getArch()) {
  default:
    llvm_unreachable("Unsupported Arch");
  case Triple::r600:
    return getEFlagsR600();
  case Triple::amdgcn:
    return getEFlagsAMDGCN();
  }
}

unsigned AMDGPUTargetELFStreamer::getEFlagsR600() {
  assert(STI.getTargetTriple().getArch() == Triple::r600);

  return getElfMach(STI.getCPU());
}

unsigned AMDGPUTargetELFStreamer::getEFlagsAMDGCN() {
  assert(STI.getTargetTriple().isAMDGCN());

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend. Main symbols: `AMDGPUTargetELFStreamer::getEFlags`, `AMDGPUTargetELFStreamer::getEFlagsR600`, `AMDGPUTargetELFStreamer::getEFlagsAMDGCN`.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。 主要符号：`AMDGPUTargetELFStreamer::getEFlags`, `AMDGPUTargetELFStreamer::getEFlagsR600`, `AMDGPUTargetELFStreamer::getEFlagsAMDGCN`。

### Lines 850-891: Switch-based control flow
```cpp
  switch (STI.getTargetTriple().getOS()) {
  default:
    // TODO: Why are some tests have "mingw" listed as OS?
    // llvm_unreachable("Unsupported OS");
  case Triple::UnknownOS:
    return getEFlagsUnknownOS();
  case Triple::AMDHSA:
    return getEFlagsAMDHSA();
  case Triple::AMDPAL:
    return getEFlagsAMDPAL();
  case Triple::Mesa3D:
    return getEFlagsMesa3D();
  }
}

unsigned AMDGPUTargetELFStreamer::getEFlagsUnknownOS() {
  // TODO: Why are some tests have "mingw" listed as OS?
  // assert(STI.getTargetTriple().getOS() == Triple::UnknownOS);

  return getEFlagsV3();
}

unsigned AMDGPUTargetELFStreamer::getEFlagsAMDHSA() {
  assert(isHsaAbi(STI));

  if (CodeObjectVersion >= 6)
    return getEFlagsV6();
  return getEFlagsV4();
}

unsigned AMDGPUTargetELFStreamer::getEFlagsAMDPAL() {
  assert(STI.getTargetTriple().getOS() == Triple::AMDPAL);

  return getEFlagsV3();
}

unsigned AMDGPUTargetELFStreamer::getEFlagsMesa3D() {
  assert(STI.getTargetTriple().getOS() == Triple::Mesa3D);

  return getEFlagsV3();
}

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `AMDGPUTargetELFStreamer::getEFlagsUnknownOS`, `AMDGPUTargetELFStreamer::getEFlagsAMDHSA`, `AMDGPUTargetELFStreamer::getEFlagsAMDPAL`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`AMDGPUTargetELFStreamer::getEFlagsUnknownOS`, `AMDGPUTargetELFStreamer::getEFlagsAMDHSA`, `AMDGPUTargetELFStreamer::getEFlagsAMDPAL`。

### Lines 892-935: Implements AMDGPUTargetELFStreamer::getEFlagsV3
```cpp
unsigned AMDGPUTargetELFStreamer::getEFlagsV3() {
  unsigned EFlagsV3 = 0;

  // mach.
  EFlagsV3 |= getElfMach(STI.getCPU());

  // xnack.
  if (getTargetID()->isXnackOnOrAny())
    EFlagsV3 |= ELF::EF_AMDGPU_FEATURE_XNACK_V3;
  // sramecc.
  if (getTargetID()->isSramEccOnOrAny())
    EFlagsV3 |= ELF::EF_AMDGPU_FEATURE_SRAMECC_V3;

  return EFlagsV3;
}

unsigned AMDGPUTargetELFStreamer::getEFlagsV4() {
  unsigned EFlagsV4 = 0;

  // mach.
  EFlagsV4 |= getElfMach(STI.getCPU());

  // xnack.
  switch (getTargetID()->getXnackSetting()) {
  case AMDGPU::IsaInfo::TargetIDSetting::Unsupported:
    EFlagsV4 |= ELF::EF_AMDGPU_FEATURE_XNACK_UNSUPPORTED_V4;
    break;
  case AMDGPU::IsaInfo::TargetIDSetting::Any:
    EFlagsV4 |= ELF::EF_AMDGPU_FEATURE_XNACK_ANY_V4;
    break;
  case AMDGPU::IsaInfo::TargetIDSetting::Off:
    EFlagsV4 |= ELF::EF_AMDGPU_FEATURE_XNACK_OFF_V4;
    break;
  case AMDGPU::IsaInfo::TargetIDSetting::On:
    EFlagsV4 |= ELF::EF_AMDGPU_FEATURE_XNACK_ON_V4;
    break;
  }
  // sramecc.
  switch (getTargetID()->getSramEccSetting()) {
  case AMDGPU::IsaInfo::TargetIDSetting::Unsupported:
    EFlagsV4 |= ELF::EF_AMDGPU_FEATURE_SRAMECC_UNSUPPORTED_V4;
    break;
  case AMDGPU::IsaInfo::TargetIDSetting::Any:
    EFlagsV4 |= ELF::EF_AMDGPU_FEATURE_SRAMECC_ANY_V4;
```
**EN:** This section contains concrete logic for AMDGPUTargetELFStreamer::getEFlagsV3. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPUTargetELFStreamer::getEFlagsV3`, `AMDGPUTargetELFStreamer::getEFlagsV4`.
**CN:** 本节包含与 AMDGPUTargetELFStreamer::getEFlagsV3 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPUTargetELFStreamer::getEFlagsV3`, `AMDGPUTargetELFStreamer::getEFlagsV4`。

### Lines 936-979: Switch-based control flow
```cpp
    break;
  case AMDGPU::IsaInfo::TargetIDSetting::Off:
    EFlagsV4 |= ELF::EF_AMDGPU_FEATURE_SRAMECC_OFF_V4;
    break;
  case AMDGPU::IsaInfo::TargetIDSetting::On:
    EFlagsV4 |= ELF::EF_AMDGPU_FEATURE_SRAMECC_ON_V4;
    break;
  }

  return EFlagsV4;
}

unsigned AMDGPUTargetELFStreamer::getEFlagsV6() {
  unsigned Flags = getEFlagsV4();

  unsigned Version = ForceGenericVersion;
  if (!Version) {
    switch (parseArchAMDGCN(STI.getCPU())) {
    case AMDGPU::GK_GFX9_GENERIC:
      Version = GenericVersion::GFX9;
      break;
    case AMDGPU::GK_GFX9_4_GENERIC:
      Version = GenericVersion::GFX9_4;
      break;
    case AMDGPU::GK_GFX10_1_GENERIC:
      Version = GenericVersion::GFX10_1;
      break;
    case AMDGPU::GK_GFX10_3_GENERIC:
      Version = GenericVersion::GFX10_3;
      break;
    case AMDGPU::GK_GFX11_GENERIC:
      Version = GenericVersion::GFX11;
      break;
    case AMDGPU::GK_GFX12_GENERIC:
      Version = GenericVersion::GFX12;
      break;
    case AMDGPU::GK_GFX12_5_GENERIC:
      Version = GenericVersion::GFX12_5;
      break;
    default:
      break;
    }
  }

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `AMDGPUTargetELFStreamer::getEFlagsV6`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`AMDGPUTargetELFStreamer::getEFlagsV6`。

### Lines 980-1020: Conditional logic and checks
```cpp
  // Versions start at 1.
  if (Version) {
    if (Version > ELF::EF_AMDGPU_GENERIC_VERSION_MAX)
      report_fatal_error("Cannot encode generic code object version " +
                         Twine(Version) +
                         " - no ELF flag can represent this version!");
    Flags |= (Version << ELF::EF_AMDGPU_GENERIC_VERSION_OFFSET);
  }

  return Flags;
}

void AMDGPUTargetELFStreamer::EmitDirectiveAMDGCNTarget() {}

void AMDGPUTargetELFStreamer::EmitAMDKernelCodeT(AMDGPUMCKernelCodeT &Header) {
  MCStreamer &OS = getStreamer();
  OS.pushSection();
  Header.EmitKernelCodeT(OS, getContext());
  OS.popSection();
}

void AMDGPUTargetELFStreamer::EmitAMDGPUSymbolType(StringRef SymbolName,
                                                   unsigned Type) {
  auto *Symbol = static_cast<MCSymbolELF *>(
      getStreamer().getContext().getOrCreateSymbol(SymbolName));
  Symbol->setType(Type);
}

void AMDGPUTargetELFStreamer::emitAMDGPULDS(MCSymbol *Symbol, unsigned Size,
                                            Align Alignment) {
  auto *SymbolELF = static_cast<MCSymbolELF *>(Symbol);
  SymbolELF->setType(ELF::STT_OBJECT);

  if (!SymbolELF->isBindingSet())
    SymbolELF->setBinding(ELF::STB_GLOBAL);

  if (SymbolELF->declareCommon(Size, Alignment)) {
    report_fatal_error("Symbol: " + Symbol->getName() +
                       " redeclared as different type");
  }

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `AMDGPUTargetELFStreamer::EmitDirectiveAMDGCNTarget`, `AMDGPUTargetELFStreamer::EmitAMDKernelCodeT`, `AMDGPUTargetELFStreamer::EmitAMDGPUSymbolType`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`AMDGPUTargetELFStreamer::EmitDirectiveAMDGCNTarget`, `AMDGPUTargetELFStreamer::EmitAMDKernelCodeT`, `AMDGPUTargetELFStreamer::EmitAMDGPUSymbolType`。

### Lines 1021-1057: Declares setIndex
```cpp
  SymbolELF->setIndex(ELF::SHN_AMDGPU_LDS);
  SymbolELF->setSize(MCConstantExpr::create(Size, getContext()));
}

bool AMDGPUTargetELFStreamer::EmitISAVersion() {
  // Create two labels to mark the beginning and end of the desc field
  // and a MCExpr to calculate the size of the desc field.
  auto &Context = getContext();
  auto *DescBegin = Context.createTempSymbol();
  auto *DescEnd = Context.createTempSymbol();
  auto *DescSZ = MCBinaryExpr::createSub(
      MCSymbolRefExpr::create(DescEnd, Context),
      MCSymbolRefExpr::create(DescBegin, Context), Context);

  EmitNote(ElfNote::NoteNameV2, DescSZ, ELF::NT_AMD_HSA_ISA_NAME,
           [&](MCELFStreamer &OS) {
             OS.emitLabel(DescBegin);

             SmallString<32> Str;
             raw_svector_ostream StrOS(Str);
             StrOS << *getTargetID();

             OS.emitBytes(StrOS.str());
             OS.emitLabel(DescEnd);
           });
  return true;
}

bool AMDGPUTargetELFStreamer::EmitHSAMetadata(msgpack::Document &HSAMetadataDoc,
                                              bool Strict) {
  HSAMD::V3::MetadataVerifier Verifier(Strict);
  if (!Verifier.verify(HSAMetadataDoc.getRoot()))
    return false;

  std::string HSAMetadataString;
  HSAMetadataDoc.writeToBlob(HSAMetadataString);

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend. Main symbols: `MCConstantExpr::create`, `AMDGPUTargetELFStreamer::EmitISAVersion`, `MCBinaryExpr::createSub`.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。 主要符号：`MCConstantExpr::create`, `AMDGPUTargetELFStreamer::EmitISAVersion`, `MCBinaryExpr::createSub`。

### Lines 1058-1101: Declares getContext
```cpp
  // Create two labels to mark the beginning and end of the desc field
  // and a MCExpr to calculate the size of the desc field.
  auto &Context = getContext();
  auto *DescBegin = Context.createTempSymbol();
  auto *DescEnd = Context.createTempSymbol();
  auto *DescSZ = MCBinaryExpr::createSub(
      MCSymbolRefExpr::create(DescEnd, Context),
      MCSymbolRefExpr::create(DescBegin, Context), Context);

  EmitNote(ElfNote::NoteNameV3, DescSZ, ELF::NT_AMDGPU_METADATA,
           [&](MCELFStreamer &OS) {
             OS.emitLabel(DescBegin);
             OS.emitBytes(HSAMetadataString);
             OS.emitLabel(DescEnd);
           });
  return true;
}

bool AMDGPUTargetELFStreamer::EmitCodeEnd(const MCSubtargetInfo &STI) {
  const uint32_t Encoded_s_code_end = 0xbf9f0000;
  const uint32_t Encoded_s_nop = 0xbf800000;
  uint32_t Encoded_pad = Encoded_s_code_end;

  // Instruction cache line size in bytes.
  const unsigned Log2CacheLineSize = AMDGPU::isGFX11Plus(STI) ? 7 : 6;
  const unsigned CacheLineSize = 1u << Log2CacheLineSize;

  // Extra padding amount in bytes to support prefetch mode 3.
  unsigned FillSize = 3 * CacheLineSize;

  if (AMDGPU::isGFX90A(STI)) {
    Encoded_pad = Encoded_s_nop;
    FillSize = 16 * CacheLineSize;
  }

  MCStreamer &OS = getStreamer();
  OS.pushSection();
  OS.emitValueToAlignment(Align(CacheLineSize), Encoded_pad, 4);
  for (unsigned I = 0; I < FillSize; I += 4)
    OS.emitInt32(Encoded_pad);
  OS.popSection();
  return true;
}

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend. Main symbols: `MCBinaryExpr::createSub`, `MCSymbolRefExpr::create`, `AMDGPUTargetELFStreamer::EmitCodeEnd`.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。 主要符号：`MCBinaryExpr::createSub`, `MCSymbolRefExpr::create`, `AMDGPUTargetELFStreamer::EmitCodeEnd`。

### Lines 1102-1142: Implements AMDGPUTargetELFStreamer::EmitAmdhsaKernelDescriptor
```cpp
void AMDGPUTargetELFStreamer::EmitAmdhsaKernelDescriptor(
    const MCSubtargetInfo &STI, StringRef KernelName,
    const MCKernelDescriptor &KernelDescriptor, const MCExpr *NextVGPR,
    const MCExpr *NextSGPR, const MCExpr *ReserveVCC,
    const MCExpr *ReserveFlatScr) {
  auto &Streamer = getStreamer();
  auto &Context = Streamer.getContext();

  auto *KernelCodeSymbol =
      static_cast<MCSymbolELF *>(Context.getOrCreateSymbol(Twine(KernelName)));
  auto *KernelDescriptorSymbol = static_cast<MCSymbolELF *>(
      Context.getOrCreateSymbol(Twine(KernelName) + Twine(".kd")));

  // Copy kernel descriptor symbol's binding, other and visibility from the
  // kernel code symbol.
  KernelDescriptorSymbol->setBinding(KernelCodeSymbol->getBinding());
  KernelDescriptorSymbol->setOther(KernelCodeSymbol->getOther());
  KernelDescriptorSymbol->setVisibility(KernelCodeSymbol->getVisibility());
  // Kernel descriptor symbol's type and size are fixed.
  KernelDescriptorSymbol->setType(ELF::STT_OBJECT);
  KernelDescriptorSymbol->setSize(
      MCConstantExpr::create(sizeof(amdhsa::kernel_descriptor_t), Context));

  // The visibility of the kernel code symbol must be protected or less to allow
  // static relocations from the kernel descriptor to be used.
  if (KernelCodeSymbol->getVisibility() == ELF::STV_DEFAULT)
    KernelCodeSymbol->setVisibility(ELF::STV_PROTECTED);

  Streamer.emitLabel(KernelDescriptorSymbol);
  Streamer.emitValue(
      KernelDescriptor.group_segment_fixed_size,
      sizeof(amdhsa::kernel_descriptor_t::group_segment_fixed_size));
  Streamer.emitValue(
      KernelDescriptor.private_segment_fixed_size,
      sizeof(amdhsa::kernel_descriptor_t::private_segment_fixed_size));
  Streamer.emitValue(KernelDescriptor.kernarg_size,
                     sizeof(amdhsa::kernel_descriptor_t::kernarg_size));

  for (uint32_t i = 0; i < sizeof(amdhsa::kernel_descriptor_t::reserved0); ++i)
    Streamer.emitInt8(0u);

```
**EN:** This section contains concrete logic for AMDGPUTargetELFStreamer::EmitAmdhsaKernelDescriptor. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPUTargetELFStreamer::EmitAmdhsaKernelDescriptor`, `MCConstantExpr::create`.
**CN:** 本节包含与 AMDGPUTargetELFStreamer::EmitAmdhsaKernelDescriptor 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPUTargetELFStreamer::EmitAmdhsaKernelDescriptor`, `MCConstantExpr::create`。

### Lines 1143-1181: Defines emitValue
```cpp
  // FIXME: Remove the use of VK_AMDGPU_REL64 in the expression below. The
  // expression being created is:
  //   (start of kernel code) - (start of kernel descriptor)
  // It implies R_AMDGPU_REL64, but ends up being R_AMDGPU_ABS64.
  Streamer.emitValue(
      MCBinaryExpr::createSub(
          MCSymbolRefExpr::create(KernelCodeSymbol, AMDGPUMCExpr::S_REL64,
                                  Context),
          MCSymbolRefExpr::create(KernelDescriptorSymbol, Context), Context),
      sizeof(amdhsa::kernel_descriptor_t::kernel_code_entry_byte_offset));
  for (uint32_t i = 0; i < sizeof(amdhsa::kernel_descriptor_t::reserved1); ++i)
    Streamer.emitInt8(0u);
  Streamer.emitValue(KernelDescriptor.compute_pgm_rsrc3,
                     sizeof(amdhsa::kernel_descriptor_t::compute_pgm_rsrc3));
  Streamer.emitValue(KernelDescriptor.compute_pgm_rsrc1,
                     sizeof(amdhsa::kernel_descriptor_t::compute_pgm_rsrc1));
  Streamer.emitValue(KernelDescriptor.compute_pgm_rsrc2,
                     sizeof(amdhsa::kernel_descriptor_t::compute_pgm_rsrc2));
  Streamer.emitValue(
      KernelDescriptor.kernel_code_properties,
      sizeof(amdhsa::kernel_descriptor_t::kernel_code_properties));
  Streamer.emitValue(KernelDescriptor.kernarg_preload,
                     sizeof(amdhsa::kernel_descriptor_t::kernarg_preload));
  for (uint32_t i = 0; i < sizeof(amdhsa::kernel_descriptor_t::reserved3); ++i)
    Streamer.emitInt8(0u);
}

void AMDGPUTargetELFStreamer::emitAMDGPUInfo(
    const AMDGPU::InfoSectionData &Data) {
  MCELFStreamer &S = getStreamer();
  MCContext &Context = S.getContext();

  StringTableBuilder StrTab(StringTableBuilder::ELF);
  auto getOrAddString = [&](StringRef Str) -> uint32_t {
    if (Str.empty())
      return UINT32_MAX;
    return StrTab.add(Str);
  };

```
**EN:** This section contains concrete logic for emitValue. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `MCBinaryExpr::createSub`, `MCSymbolRefExpr::create`, `AMDGPUTargetELFStreamer::emitAMDGPUInfo`.
**CN:** 本节包含与 emitValue 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`MCBinaryExpr::createSub`, `MCSymbolRefExpr::create`, `AMDGPUTargetELFStreamer::emitAMDGPUInfo`。

### Lines 1182-1223: Declares pushSection
```cpp
  auto EmitU32Entry = [&](AMDGPU::InfoKind Kind, uint32_t Val) {
    S.emitInt8(static_cast<uint8_t>(Kind));
    S.emitInt8(4);
    S.emitInt32(Val);
  };
  auto EmitSymEntry = [&](AMDGPU::InfoKind Kind, MCSymbol *Sym) {
    S.emitInt8(static_cast<uint8_t>(Kind));
    S.emitInt8(8);
    S.emitValue(MCSymbolRefExpr::create(Sym, Context), 8);
  };

  S.pushSection();
  MCSectionELF *InfoSec = Context.getELFSection(
      ".amdgpu.info", ELF::SHT_PROGBITS, ELF::SHF_EXCLUDE);
  S.switchSection(InfoSec);

  forEachInfoScope(Data, [&](MCSymbol *Sym, const AMDGPU::FuncInfo *Info,
                             ArrayRef<MCSymbol *> Uses,
                             ArrayRef<MCSymbol *> Calls,
                             ArrayRef<StringRef> IndirectCallTypeIds,
                             ArrayRef<StringRef> TypeIds) {
    EmitSymEntry(AMDGPU::InfoKind::INFO_FUNC, Sym);

    if (Info) {
      AMDGPU::FuncInfoFlags Flags{};
      if (Info->UsesVCC)
        Flags |= AMDGPU::FuncInfoFlags::FUNC_USES_VCC;
      if (Info->UsesFlatScratch)
        Flags |= AMDGPU::FuncInfoFlags::FUNC_USES_FLAT_SCRATCH;
      if (Info->HasDynStack)
        Flags |= AMDGPU::FuncInfoFlags::FUNC_HAS_DYN_STACK;
      EmitU32Entry(AMDGPU::InfoKind::INFO_FLAGS, llvm::to_underlying(Flags));
      EmitU32Entry(AMDGPU::InfoKind::INFO_NUM_SGPR, Info->NumSGPR);
      EmitU32Entry(AMDGPU::InfoKind::INFO_NUM_VGPR, Info->NumArchVGPR);
      // INFO_NUM_AGPR is only emitted when the function actually uses AGPRs,
      // since AGPRs are not available on all architectures.
      if (Info->NumAccVGPR)
        EmitU32Entry(AMDGPU::InfoKind::INFO_NUM_AGPR, Info->NumAccVGPR);
      EmitU32Entry(AMDGPU::InfoKind::INFO_PRIVATE_SEGMENT_SIZE,
                   Info->PrivateSegmentSize);
    }

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend. Main symbols: `MCSymbolRefExpr::create`, `llvm::to_underlying`.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。 主要符号：`MCSymbolRefExpr::create`, `llvm::to_underlying`。

### Lines 1224-1248: Conditional logic and checks
```cpp
    for (MCSymbol *Res : Uses)
      EmitSymEntry(AMDGPU::InfoKind::INFO_USE, Res);
    for (MCSymbol *Dst : Calls)
      EmitSymEntry(AMDGPU::InfoKind::INFO_CALL, Dst);
    for (StringRef TypeId : IndirectCallTypeIds) {
      EmitU32Entry(AMDGPU::InfoKind::INFO_INDIRECT_CALL,
                   getOrAddString(TypeId));
    }
    for (StringRef TypeId : TypeIds)
      EmitU32Entry(AMDGPU::InfoKind::INFO_TYPEID, getOrAddString(TypeId));
  });

  if (!StrTab.empty()) {
    StrTab.finalizeInOrder();
    MCSectionELF *Sec = Context.getELFSection(".amdgpu.strtab", ELF::SHT_STRTAB,
                                              ELF::SHF_EXCLUDE);
    S.switchSection(Sec);
    SmallString<128> Buf;
    raw_svector_ostream OS(Buf);
    StrTab.write(OS);
    S.emitBytes(Buf);
  }

  S.popSection();
}
```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

## Key Concepts / 关键概念
- **Language / 语言**: C++ source
- **Primary symbols / 主要符号**: `cl::desc`, `cl::init`, `AMDGPUTargetStreamer::EmitHSAMetadataV3`, `AMDGPUTargetStreamer::getArchNameFromElfMach`, `AMDGPUTargetStreamer::getElfMach`, `AMDGPUTargetAsmStreamer::AMDGPUTargetAsmStreamer`
- **Main themes / 核心主题**: subtarget modeling / 子目标建模; assembly handling / 汇编处理
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"AMDGPUTargetStreamer.h"`
- `"AMDGPUMCExpr.h"`
- `"AMDGPUMCKernelDescriptor.h"`
- `"AMDGPUMCTargetDesc.h"`
- `"AMDGPUPTNote.h"`
- `"Utils/AMDGPUBaseInfo.h"`
- `"Utils/AMDKernelCodeTUtils.h"`
- `"llvm/BinaryFormat/AMDGPUMetadataVerifier.h"`
- `"llvm/BinaryFormat/ELF.h"`
- `"llvm/MC/MCAsmInfo.h"`
- `"llvm/MC/MCAssembler.h"`
- `"llvm/MC/MCContext.h"`
- `"llvm/MC/MCELFObjectWriter.h"`
- `"llvm/MC/MCELFStreamer.h"`
- `"llvm/MC/MCSubtargetInfo.h"`
- `"llvm/MC/StringTableBuilder.h"`
- `"llvm/Support/AMDGPUMetadata.h"`
- `"llvm/Support/AMDGPUObjLinkingInfo.h"`
- `"llvm/Support/AMDHSAKernelDescriptor.h"`
- `"llvm/Support/CommandLine.h"`
- ... and 2 more direct dependencies / 以及另外 2 个直接依赖
