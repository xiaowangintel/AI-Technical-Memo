# AArch64WinCOFFObjectWriter.cpp — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/AArch64/MCTargetDesc/AArch64WinCOFFObjectWriter.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This file covers AArch64 Windows COFF Object Writer C++. / 该文件实现 AArch64 后端中的AArch64 后端逻辑。
## Line-by-Line Analysis / 逐行分析
### Lines 1-27: Documented code section
```cpp
//= AArch64WinCOFFObjectWriter.cpp - AArch64 Windows COFF Object Writer C++ =//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===---------------------------------------------------------------------===//

#include "AArch64MCTargetDesc.h"
#include "MCTargetDesc/AArch64FixupKinds.h"
#include "MCTargetDesc/AArch64MCAsmInfo.h"
#include "llvm/ADT/Twine.h"
#include "llvm/BinaryFormat/COFF.h"
#include "llvm/MC/MCAsmBackend.h"
#include "llvm/MC/MCContext.h"
#include "llvm/MC/MCExpr.h"
#include "llvm/MC/MCFixup.h"
#include "llvm/MC/MCObjectWriter.h"
#include "llvm/MC/MCValue.h"
#include "llvm/MC/MCWinCOFFObjectWriter.h"
#include "llvm/Support/Casting.h"
#include "llvm/Support/ErrorHandling.h"
#include <cassert>

using namespace llvm;

namespace {
```
**EN:** This comment block marks a new section and frames how the following implementation should be read.  
**CN:** 该注释块标记了新的代码章节，并为理解后续实现提供上下文。
### Lines 28-45: Class AArch64WinCOFFObjectWriter
```cpp

class AArch64WinCOFFObjectWriter : public MCWinCOFFObjectTargetWriter {
public:
  AArch64WinCOFFObjectWriter(const Triple &TheTriple)
      : MCWinCOFFObjectTargetWriter(TheTriple.isWindowsArm64EC()
                                        ? COFF::IMAGE_FILE_MACHINE_ARM64EC
                                        : COFF::IMAGE_FILE_MACHINE_ARM64) {}

  ~AArch64WinCOFFObjectWriter() override = default;

  unsigned getRelocType(MCContext &Ctx, const MCValue &Target,
                        const MCFixup &Fixup, bool IsCrossSection,
                        const MCAsmBackend &MAB) const override;

  bool recordRelocation(const MCFixup &) const override;
};

} // end anonymous namespace
```
**EN:** This block defines AArch64WinCOFFObjectWriter, packaging state and behavior that the file reuses for AArch64 backend logic.  
**CN:** 该代码块定义 AArch64WinCOFFObjectWriter，把 AArch64 后端逻辑 所需的状态与行为封装在一起供后续复用。
### Lines 46-66: Function AArch64WinCOFFObjectWriter::getRelocType
```cpp

unsigned AArch64WinCOFFObjectWriter::getRelocType(
    MCContext &Ctx, const MCValue &Target, const MCFixup &Fixup,
    bool IsCrossSection, const MCAsmBackend &MAB) const {
  unsigned FixupKind = Fixup.getKind();
  bool PCRel = Fixup.isPCRel();
  if (IsCrossSection) {
    // IMAGE_REL_ARM64_REL64 does not exist. We treat FK_Data_8 as FK_PCRel_4 so
    // that .xword a-b can lower to IMAGE_REL_ARM64_REL32. This allows generic
    // instrumentation to not bother with the COFF limitation. A negative value
    // needs attention.
    if (PCRel || (FixupKind != FK_Data_4 && FixupKind != FK_Data_8)) {
      Ctx.reportError(Fixup.getLoc(), "Cannot represent this expression");
      return COFF::IMAGE_REL_ARM64_ADDR32;
    }
    FixupKind = FK_Data_4;
    PCRel = true;
  }

  auto Spec = Target.getSpecifier();
  const MCExpr *Expr = Fixup.getValue();
```
**EN:** This block implements AArch64WinCOFFObjectWriter::getRelocType, advancing the file's AArch64 backend logic flow with AArch64-specific decisions and data movement.  
**CN:** 该代码块实现 AArch64WinCOFFObjectWriter::getRelocType，通过 AArch64 专用的决策与数据处理推进本文件的AArch64 后端逻辑流程。
### Lines 67-82: Core AArch64 backend logic
```cpp

  if (auto *A64E = dyn_cast<MCSpecifierExpr>(Expr)) {
    AArch64::Specifier Spec = A64E->getSpecifier();
    switch (AArch64::getSymbolLoc(Spec)) {
    case AArch64::S_ABS:
    case AArch64::S_SECREL:
      // Supported
      break;
    default:
      Ctx.reportError(Fixup.getLoc(),
                      "relocation specifier " +
                          AArch64::getSpecifierName(A64E->getSpecifier()) +
                          " unsupported on COFF targets");
      return COFF::IMAGE_REL_ARM64_ABSOLUTE; // Dummy return value
    }
  }
```
**EN:** This block continues the file's main AArch64 backend logic logic and connects local helpers with the wider AArch64 backend.  
**CN:** 该代码块延续本文件的AArch64 后端逻辑主线逻辑，并把本地辅助实现连接到更广泛的 AArch64 后端中。
### Lines 83-107: Core AArch64 backend logic
```cpp

  switch (FixupKind) {
  default: {
    if (auto *A64E = dyn_cast<MCSpecifierExpr>(Expr)) {
      Ctx.reportError(Fixup.getLoc(),
                      "relocation specifier " +
                          AArch64::getSpecifierName(A64E->getSpecifier()) +
                          " unsupported on COFF targets");
    } else {
      MCFixupKindInfo Info = MAB.getFixupKindInfo(Fixup.getKind());
      Ctx.reportError(Fixup.getLoc(), Twine("relocation type ") + Info.Name +
                                          " unsupported on COFF targets");
    }
    return COFF::IMAGE_REL_ARM64_ABSOLUTE; // Dummy return value
  }

  case FK_Data_4:
    if (PCRel)
      return COFF::IMAGE_REL_ARM64_REL32;
    switch (Spec) {
    default:
      return COFF::IMAGE_REL_ARM64_ADDR32;
    case MCSymbolRefExpr::VK_COFF_IMGREL32:
      return COFF::IMAGE_REL_ARM64_ADDR32NB;
    }
```
**EN:** This block continues the file's main AArch64 backend logic logic and connects local helpers with the wider AArch64 backend.  
**CN:** 该代码块延续本文件的AArch64 后端逻辑主线逻辑，并把本地辅助实现连接到更广泛的 AArch64 后端中。
### Lines 108-126: Core AArch64 backend logic
```cpp

  case FK_Data_8:
    return COFF::IMAGE_REL_ARM64_ADDR64;

  case FK_SecRel_2:
    return COFF::IMAGE_REL_ARM64_SECTION;

  case FK_SecRel_4:
    return COFF::IMAGE_REL_ARM64_SECREL;

  case AArch64::fixup_aarch64_add_imm12:
    if (auto *A64E = dyn_cast<MCSpecifierExpr>(Expr)) {
      AArch64::Specifier Spec = A64E->getSpecifier();
      if (Spec == AArch64::S_SECREL_LO12)
        return COFF::IMAGE_REL_ARM64_SECREL_LOW12A;
      if (Spec == AArch64::S_SECREL_HI12)
        return COFF::IMAGE_REL_ARM64_SECREL_HIGH12A;
    }
    return COFF::IMAGE_REL_ARM64_PAGEOFFSET_12A;
```
**EN:** This block continues the file's main AArch64 backend logic logic and connects local helpers with the wider AArch64 backend.  
**CN:** 该代码块延续本文件的AArch64 后端逻辑主线逻辑，并把本地辅助实现连接到更广泛的 AArch64 后端中。
### Lines 127-150: Core AArch64 backend logic
```cpp

  case AArch64::fixup_aarch64_ldst_imm12_scale1:
  case AArch64::fixup_aarch64_ldst_imm12_scale2:
  case AArch64::fixup_aarch64_ldst_imm12_scale4:
  case AArch64::fixup_aarch64_ldst_imm12_scale8:
  case AArch64::fixup_aarch64_ldst_imm12_scale16:
    if (auto *A64E = dyn_cast<MCSpecifierExpr>(Expr)) {
      AArch64::Specifier Spec = A64E->getSpecifier();
      if (Spec == AArch64::S_SECREL_LO12)
        return COFF::IMAGE_REL_ARM64_SECREL_LOW12L;
    }
    return COFF::IMAGE_REL_ARM64_PAGEOFFSET_12L;

  case AArch64::fixup_aarch64_pcrel_adr_imm21:
    return COFF::IMAGE_REL_ARM64_REL21;

  case AArch64::fixup_aarch64_pcrel_adrp_imm21:
    return COFF::IMAGE_REL_ARM64_PAGEBASE_REL21;

  case AArch64::fixup_aarch64_pcrel_branch14:
    return COFF::IMAGE_REL_ARM64_BRANCH14;

  case AArch64::fixup_aarch64_pcrel_branch19:
    return COFF::IMAGE_REL_ARM64_BRANCH19;
```
**EN:** This block continues the file's main AArch64 backend logic logic and connects local helpers with the wider AArch64 backend.  
**CN:** 该代码块延续本文件的AArch64 后端逻辑主线逻辑，并把本地辅助实现连接到更广泛的 AArch64 后端中。
### Lines 151-165: Function AArch64WinCOFFObjectWriter::recordRelocation
```cpp

  case AArch64::fixup_aarch64_pcrel_branch26:
  case AArch64::fixup_aarch64_pcrel_call26:
    return COFF::IMAGE_REL_ARM64_BRANCH26;
  }
}

bool AArch64WinCOFFObjectWriter::recordRelocation(const MCFixup &Fixup) const {
  return true;
}

std::unique_ptr<MCObjectTargetWriter>
llvm::createAArch64WinCOFFObjectWriter(const Triple &TheTriple) {
  return std::make_unique<AArch64WinCOFFObjectWriter>(TheTriple);
}
```
**EN:** This block implements AArch64WinCOFFObjectWriter::recordRelocation, advancing the file's AArch64 backend logic flow with AArch64-specific decisions and data movement.  
**CN:** 该代码块实现 AArch64WinCOFFObjectWriter::recordRelocation，通过 AArch64 专用的决策与数据处理推进本文件的AArch64 后端逻辑流程。
## Key Concepts / 关键概念
- **EN:** AArch64 target backend structure **CN:** AArch64 目标后端结构
## Dependencies / 依赖关系
- **EN:** Target-local includes: AArch64MCTargetDesc.h, MCTargetDesc/AArch64FixupKinds.h, MCTargetDesc/AArch64MCAsmInfo.h **CN:** 目标本地依赖：AArch64MCTargetDesc.h, MCTargetDesc/AArch64FixupKinds.h, MCTargetDesc/AArch64MCAsmInfo.h
- **EN:** Core LLVM interfaces: llvm/ADT/Twine.h, llvm/BinaryFormat/COFF.h, llvm/MC/MCAsmBackend.h, llvm/MC/MCContext.h, llvm/MC/MCExpr.h, llvm/MC/MCFixup.h, llvm/MC/MCObjectWriter.h, llvm/MC/MCValue.h, llvm/MC/MCWinCOFFObjectWriter.h, llvm/Support/Casting.h ... **CN:** 核心 LLVM 接口：llvm/ADT/Twine.h, llvm/BinaryFormat/COFF.h, llvm/MC/MCAsmBackend.h, llvm/MC/MCContext.h, llvm/MC/MCExpr.h, llvm/MC/MCFixup.h, llvm/MC/MCObjectWriter.h, llvm/MC/MCValue.h, llvm/MC/MCWinCOFFObjectWriter.h, llvm/Support/Casting.h ...
- **EN:** Standard-library support: cassert **CN:** 标准库支持：cassert
- **EN:** Closely connected with neighboring AArch64 backend components responsible for AArch64 backend logic. **CN:** 与周边负责AArch64 后端逻辑的 AArch64 后端组件紧密协作。
