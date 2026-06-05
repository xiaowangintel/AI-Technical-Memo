# PPCXCOFFObjectWriter.cpp — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/PowerPC/MCTargetDesc/PPCXCOFFObjectWriter.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file provides MC layer support for the backend for the PowerPC backend.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/PowerPC/MCTargetDesc/PPCXCOFFObjectWriter.cpp`，主要负责 PowerPC 后端的该后端的 MC 层支持。 文件内容以具体实现、辅助函数和后端决策逻辑为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7

```cpp
//===-- PPCXCOFFObjectWriter.cpp - PowerPC XCOFF Writer -------------------===//
//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。

### Lines 8-14

```cpp
//===----------------------------------------------------------------------===//

#include "MCTargetDesc/PPCFixupKinds.h"
#include "MCTargetDesc/PPCMCTargetDesc.h"
#include "PPCMCAsmInfo.h"
#include "llvm/BinaryFormat/XCOFF.h"
#include "llvm/MC/MCFixup.h"
```
- **EN**: Pulls in direct dependencies required by this MC layer support for the backend, so later declarations can reuse LLVM infrastructure and target-specific helpers. The logic interacts with LLVM's MC layer.
- **CN**: 这一段引入该该后端的 MC 层支持所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 相关逻辑会与 LLVM 的 MC 层交互。

### Lines 15-52

```cpp
#include "llvm/MC/MCValue.h"
#include "llvm/MC/MCXCOFFObjectWriter.h"

using namespace llvm;

namespace {
class PPCXCOFFObjectWriter : public MCXCOFFObjectTargetWriter {
  static constexpr uint8_t SignBitMask = 0x80;

public:
  PPCXCOFFObjectWriter(bool Is64Bit);

  std::pair<uint8_t, uint8_t>
  getRelocTypeAndSignSize(const MCValue &Target, const MCFixup &Fixup,
                          bool IsPCRel) const override;
};
} // end anonymous namespace

PPCXCOFFObjectWriter::PPCXCOFFObjectWriter(bool Is64Bit)
    : MCXCOFFObjectTargetWriter(Is64Bit) {}

std::unique_ptr<MCObjectTargetWriter>
llvm::createPPCXCOFFObjectWriter(bool Is64Bit) {
  return std::make_unique<PPCXCOFFObjectWriter>(Is64Bit);
}

std::pair<uint8_t, uint8_t> PPCXCOFFObjectWriter::getRelocTypeAndSignSize(
    const MCValue &Target, const MCFixup &Fixup, bool IsPCRel) const {
  const auto Specifier = Target.getSpecifier();
  // People from AIX OS team says AIX link editor does not care about
  // the sign bit in the relocation entry "most" of the time.
  // The system assembler seems to set the sign bit on relocation entry
  // based on similar property of IsPCRel. So we will do the same here.
  // TODO: More investigation on how assembler decides to set the sign
  // bit, and we might want to match that.
  const uint8_t EncodedSignednessIndicator = IsPCRel ? SignBitMask : 0u;

  // The magic number we use in SignAndSize has a strong relationship with
```
- **EN**: Pulls in direct dependencies required by this MC layer support for the backend, so later declarations can reuse LLVM infrastructure and target-specific helpers. Notable symbols in this range include `PPCXCOFFObjectWriter`, `getRelocTypeAndSignSize`, `MCXCOFFObjectTargetWriter`.
- **CN**: 这一段引入该该后端的 MC 层支持所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 该区间中较显眼的符号包括 `PPCXCOFFObjectWriter`, `getRelocTypeAndSignSize`, `MCXCOFFObjectTargetWriter`。

### Lines 53-90

```cpp
  // the corresponding MCFixupKind. In most cases, it's the MCFixupKind
  // number - 1, because SignAndSize encodes the bit length being
  // relocated minus 1.
  switch ((unsigned)Fixup.getKind()) {
  default:
    report_fatal_error("Unimplemented fixup kind.");
  case XCOFF::RelocationType::R_REF:
    return {XCOFF::RelocationType::R_REF, 0};
  case PPC::fixup_ppc_half16: {
    const uint8_t SignAndSizeForHalf16 = EncodedSignednessIndicator | 15;
    switch (Specifier) {
    default:
      report_fatal_error("Unsupported modifier for half16 fixup.");
    case PPC::S_None:
      return {XCOFF::RelocationType::R_TOC, SignAndSizeForHalf16};
    case PPC::S_U:
      return {XCOFF::RelocationType::R_TOCU, SignAndSizeForHalf16};
    case PPC::S_L:
      return {XCOFF::RelocationType::R_TOCL, SignAndSizeForHalf16};
    case PPC::S_AIX_TLSLE:
      return {XCOFF::RelocationType::R_TLS_LE, SignAndSizeForHalf16};
    case PPC::S_AIX_TLSLD:
      return {XCOFF::RelocationType::R_TLS_LD, SignAndSizeForHalf16};
    }
  } break;
  case PPC::fixup_ppc_half16ds:
  case PPC::fixup_ppc_half16dq: {
    if (IsPCRel)
      report_fatal_error("Invalid PC-relative relocation.");
    switch (Specifier) {
    default:
      llvm_unreachable("Unsupported Modifier");
    case PPC::S_None:
      return {XCOFF::RelocationType::R_TOC, 15};
    case PPC::S_L:
      return {XCOFF::RelocationType::R_TOCL, 15};
    case PPC::S_AIX_TLSLE:
      return {XCOFF::RelocationType::R_TLS_LE, 15};
```
- **EN**: Implements helper routine(s) `getKind`, `report_fatal_error`, `llvm_unreachable` for this portion of the PowerPC backend MC layer support for the backend.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 MC 层支持所需的辅助例程 `getKind`, `report_fatal_error`, `llvm_unreachable`。

### Lines 91-125

```cpp
    case PPC::S_AIX_TLSLD:
      return {XCOFF::RelocationType::R_TLS_LD, 15};
    }
  } break;
  case PPC::fixup_ppc_br24:
    // Branches are 4 byte aligned, so the 24 bits we encode in
    // the instruction actually represents a 26 bit offset.
    return {XCOFF::RelocationType::R_RBR, EncodedSignednessIndicator | 25};
  case PPC::fixup_ppc_br24abs:
    return {XCOFF::RelocationType::R_RBA, EncodedSignednessIndicator | 25};
  case FK_Data_4:
  case FK_Data_8:
    const uint8_t SignAndSizeForFKData =
        EncodedSignednessIndicator |
        ((unsigned)Fixup.getKind() == FK_Data_4 ? 31 : 63);
    switch (Specifier) {
    default:
      report_fatal_error("Unsupported modifier");
    case PPC::S_AIX_TLSGD:
      return {XCOFF::RelocationType::R_TLS, SignAndSizeForFKData};
    case PPC::S_AIX_TLSGDM:
      return {XCOFF::RelocationType::R_TLSM, SignAndSizeForFKData};
    case PPC::S_AIX_TLSIE:
      return {XCOFF::RelocationType::R_TLS_IE, SignAndSizeForFKData};
    case PPC::S_AIX_TLSLE:
      return {XCOFF::RelocationType::R_TLS_LE, SignAndSizeForFKData};
    case PPC::S_AIX_TLSLD:
      return {XCOFF::RelocationType::R_TLS_LD, SignAndSizeForFKData};
    case PPC::S_AIX_TLSML:
      return {XCOFF::RelocationType::R_TLSML, SignAndSizeForFKData};
    case PPC::S_None:
      return {XCOFF::RelocationType::R_POS, SignAndSizeForFKData};
    }
  }
}
```
- **EN**: Implements helper routine(s) `getKind`, `report_fatal_error` for this portion of the PowerPC backend MC layer support for the backend.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 MC 层支持所需的辅助例程 `getKind`, `report_fatal_error`。

## Key Concepts / 关键概念

- PowerPC backend integration / PowerPC 后端集成
- MC layer support for the backend / 该后端的 MC 层支持
- Exception-handling support / 异常处理支持

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `MCTargetDesc/PPCFixupKinds.h`
- `MCTargetDesc/PPCMCTargetDesc.h`
- `PPCMCAsmInfo.h`
- `llvm/BinaryFormat/XCOFF.h`
- `llvm/MC/MCFixup.h`
- `llvm/MC/MCValue.h`
- `llvm/MC/MCXCOFFObjectWriter.h`

### Important Collaborators / 重要协作组件

- LLVM MC layer / LLVM MC 层
- PowerPC target-specific helpers / PowerPC 目标专用辅助组件
