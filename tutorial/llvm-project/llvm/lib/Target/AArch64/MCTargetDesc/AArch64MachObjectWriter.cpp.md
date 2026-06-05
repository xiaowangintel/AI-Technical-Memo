# AArch64MachObjectWriter.cpp — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/AArch64/MCTargetDesc/AArch64MachObjectWriter.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This file covers ARM Mach Object Writer. / 该文件实现 AArch64 后端中的AArch64 后端逻辑。
## Line-by-Line Analysis / 逐行分析
### Lines 1-32: Documented code section
```cpp
//===-- AArch64MachObjectWriter.cpp - ARM Mach Object Writer --------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "MCTargetDesc/AArch64FixupKinds.h"
#include "MCTargetDesc/AArch64MCAsmInfo.h"
#include "MCTargetDesc/AArch64MCTargetDesc.h"
#include "llvm/ADT/Twine.h"
#include "llvm/BinaryFormat/MachO.h"
#include "llvm/MC/MCAsmInfo.h"
#include "llvm/MC/MCAsmInfoDarwin.h"
#include "llvm/MC/MCAssembler.h"
#include "llvm/MC/MCContext.h"
#include "llvm/MC/MCExpr.h"
#include "llvm/MC/MCFixup.h"
#include "llvm/MC/MCMachObjectWriter.h"
#include "llvm/MC/MCSection.h"
#include "llvm/MC/MCSectionMachO.h"
#include "llvm/MC/MCSymbol.h"
#include "llvm/MC/MCValue.h"
#include "llvm/Support/Casting.h"
#include "llvm/Support/MathExtras.h"
#include <cassert>
#include <cstdint>

using namespace llvm;

namespace {
```
**EN:** This comment block marks a new section and frames how the following implementation should be read.  
**CN:** 该注释块标记了新的代码章节，并为理解后续实现提供上下文。
### Lines 33-58: Class AArch64MachObjectWriter
```cpp

class AArch64MachObjectWriter : public MCMachObjectTargetWriter {
  bool getAArch64FixupKindMachOInfo(const MCFixup &Fixup, unsigned &RelocType,
                                    AArch64::Specifier Spec, unsigned &Log2Size,
                                    const MCAssembler &Asm);

public:
  AArch64MachObjectWriter(uint32_t CPUType, uint32_t CPUSubtype, bool IsILP32)
      : MCMachObjectTargetWriter(!IsILP32 /* is64Bit */, CPUType, CPUSubtype) {}

  void recordRelocation(MachObjectWriter *Writer, MCAssembler &Asm,
                        const MCFragment *Fragment, const MCFixup &Fixup,
                        MCValue Target, uint64_t &FixedValue) override;
};

} // end anonymous namespace

bool AArch64MachObjectWriter::getAArch64FixupKindMachOInfo(
    const MCFixup &Fixup, unsigned &RelocType, AArch64::Specifier Spec,
    unsigned &Log2Size, const MCAssembler &Asm) {
  RelocType = unsigned(MachO::ARM64_RELOC_UNSIGNED);
  Log2Size = ~0U;

  switch (Fixup.getKind()) {
  default:
    return false;
```
**EN:** This block defines AArch64MachObjectWriter, packaging state and behavior that the file reuses for AArch64 backend logic.  
**CN:** 该代码块定义 AArch64MachObjectWriter，把 AArch64 后端逻辑 所需的状态与行为封装在一起供后续复用。
### Lines 59-94: Core AArch64 backend logic
```cpp

  case FK_Data_1:
    Log2Size = Log2_32(1);
    return true;
  case FK_Data_2:
    Log2Size = Log2_32(2);
    return true;
  case FK_Data_4:
    Log2Size = Log2_32(4);
    if (Spec == AArch64::S_MACHO_GOT)
      RelocType = unsigned(MachO::ARM64_RELOC_POINTER_TO_GOT);
    return true;
  case FK_Data_8:
    Log2Size = Log2_32(8);
    if (Spec == AArch64::S_MACHO_GOT)
      RelocType = unsigned(MachO::ARM64_RELOC_POINTER_TO_GOT);
    return true;
  case AArch64::fixup_aarch64_add_imm12:
  case AArch64::fixup_aarch64_ldst_imm12_scale1:
  case AArch64::fixup_aarch64_ldst_imm12_scale2:
  case AArch64::fixup_aarch64_ldst_imm12_scale4:
  case AArch64::fixup_aarch64_ldst_imm12_scale8:
  case AArch64::fixup_aarch64_ldst_imm12_scale16:
    Log2Size = Log2_32(4);
    switch (Spec) {
    default:
      return false;
    case AArch64::S_MACHO_PAGEOFF:
      RelocType = unsigned(MachO::ARM64_RELOC_PAGEOFF12);
      return true;
    case AArch64::S_MACHO_GOTPAGEOFF:
      RelocType = unsigned(MachO::ARM64_RELOC_GOT_LOAD_PAGEOFF12);
      return true;
    case AArch64::S_MACHO_TLVPPAGEOFF:
      RelocType = unsigned(MachO::ARM64_RELOC_TLVP_LOAD_PAGEOFF12);
      return true;
```
**EN:** This block continues the file's main AArch64 backend logic logic and connects local helpers with the wider AArch64 backend.  
**CN:** 该代码块延续本文件的AArch64 后端逻辑主线逻辑，并把本地辅助实现连接到更广泛的 AArch64 后端中。
### Lines 95-126: Core AArch64 backend logic
```cpp
    }
  case AArch64::fixup_aarch64_pcrel_adrp_imm21:
    Log2Size = Log2_32(4);
    // This encompasses the relocation for the whole 21-bit value.
    switch (Spec) {
    default:
      reportError(Fixup.getLoc(), "ADR/ADRP relocations must be GOT relative");
      return false;
    case AArch64::S_MACHO_PAGE:
      RelocType = unsigned(MachO::ARM64_RELOC_PAGE21);
      return true;
    case AArch64::S_MACHO_GOTPAGE:
      RelocType = unsigned(MachO::ARM64_RELOC_GOT_LOAD_PAGE21);
      return true;
    case AArch64::S_MACHO_TLVPPAGE:
      RelocType = unsigned(MachO::ARM64_RELOC_TLVP_LOAD_PAGE21);
      return true;
    }
    return true;
  case AArch64::fixup_aarch64_pcrel_branch26:
  case AArch64::fixup_aarch64_pcrel_call26:
    Log2Size = Log2_32(4);
    RelocType = unsigned(MachO::ARM64_RELOC_BRANCH26);
    return true;
  }
}

static bool canUseLocalRelocation(const MCSectionMachO &Section,
                                  const MCSymbol &Symbol, unsigned Log2Size) {
  // Debug info sections can use local relocations.
  if (Section.hasAttribute(MachO::S_ATTR_DEBUG))
    return true;
```
**EN:** This block continues the file's main AArch64 backend logic logic and connects local helpers with the wider AArch64 backend.  
**CN:** 该代码块延续本文件的AArch64 后端逻辑主线逻辑，并把本地辅助实现连接到更广泛的 AArch64 后端中。
### Lines 127-160: Documented code section
```cpp

  // Otherwise, only pointer sized relocations are supported.
  if (Log2Size != 3)
    return false;

  // But only if they don't point to a few forbidden sections.
  if (!Symbol.isInSection())
    return true;
  const MCSectionMachO &RefSec =
      static_cast<MCSectionMachO &>(Symbol.getSection());
  if (RefSec.getType() == MachO::S_CSTRING_LITERALS)
    return false;

  if (RefSec.getSegmentName() == "__DATA" &&
      (RefSec.getName() == "__cfstring" ||
       RefSec.getName() == "__objc_classrefs"))
    return false;

  return true;
}

void AArch64MachObjectWriter::recordRelocation(
    MachObjectWriter *Writer, MCAssembler &Asm, const MCFragment *Fragment,
    const MCFixup &Fixup, MCValue Target, uint64_t &FixedValue) {
  unsigned IsPCRel = Fixup.isPCRel();

  // See <reloc.h>.
  uint32_t FixupOffset = Asm.getFragmentOffset(*Fragment);
  unsigned Log2Size = 0;
  int64_t Value = 0;
  unsigned Index = 0;
  unsigned Type = 0;
  unsigned Kind = Fixup.getKind();
  const MCSymbol *RelSymbol = nullptr;
```
**EN:** This comment block marks a new section and frames how the following implementation should be read.  
**CN:** 该注释块标记了新的代码章节，并为理解后续实现提供上下文。
### Lines 161-190: Core AArch64 backend logic
```cpp

  FixupOffset += Fixup.getOffset();

  // AArch64 pcrel relocation addends do not include the section offset.
  if (IsPCRel)
    FixedValue += FixupOffset;

  // ADRP fixups use relocations for the whole symbol value and only
  // put the addend in the instruction itself. Clear out any value the
  // generic code figured out from the sybmol definition.
  if (Kind == AArch64::fixup_aarch64_pcrel_adrp_imm21)
    FixedValue = 0;

  // imm19 relocations are for conditional branches, which require
  // assembler local symbols. If we got here, that's not what we have,
  // so complain loudly.
  if (Kind == AArch64::fixup_aarch64_pcrel_branch19) {
    reportError(Fixup.getLoc(), "conditional branch requires assembler-local"
                                " label. '" +
                                    Target.getAddSym()->getName() +
                                    "' is external.");
    return;
  }

  // 14-bit branch relocations should only target internal labels, and so
  // should never get here.
  if (Kind == AArch64::fixup_aarch64_pcrel_branch14) {
    reportError(Fixup.getLoc(), "Invalid relocation on conditional branch!");
    return;
  }
```
**EN:** This block continues the file's main AArch64 backend logic logic and connects local helpers with the wider AArch64 backend.  
**CN:** 该代码块延续本文件的AArch64 后端逻辑主线逻辑，并把本地辅助实现连接到更广泛的 AArch64 后端中。
### Lines 191-215: Core AArch64 backend logic
```cpp

  if (!getAArch64FixupKindMachOInfo(Fixup, Type, Target.getSpecifier(),
                                    Log2Size, Asm)) {
    reportError(Fixup.getLoc(), "unknown AArch64 fixup kind!");
    return;
  }

  Value = Target.getConstant();

  if (Target.isAbsolute()) { // constant
    // FIXME: Should this always be extern?
    // SymbolNum of 0 indicates the absolute section.
    Type = MachO::ARM64_RELOC_UNSIGNED;

    if (IsPCRel) {
      reportError(Fixup.getLoc(), "PC relative absolute relocation!");
      return;

      // FIXME: x86_64 sets the type to a branch reloc here. Should we do
      // something similar?
    }
  } else if (auto *B = Target.getSubSym()) { // A - B + constant
    const MCSymbol *A = Target.getAddSym();
    const MCSymbol *A_Base = Writer->getAtom(*A);
    const MCSymbol *B_Base = Writer->getAtom(*B);
```
**EN:** This block continues the file's main AArch64 backend logic logic and connects local helpers with the wider AArch64 backend.  
**CN:** 该代码块延续本文件的AArch64 后端逻辑主线逻辑，并把本地辅助实现连接到更广泛的 AArch64 后端中。
### Lines 216-242: Documented code section
```cpp

    // Check for "_foo@got - .", which comes through here as:
    // Ltmp0:
    //    ... _foo@got - Ltmp0
    if (Target.getSpecifier() == AArch64::S_MACHO_GOT &&
        Asm.getSymbolOffset(*B) ==
            Asm.getFragmentOffset(*Fragment) + Fixup.getOffset()) {
      // SymB is the PC, so use a PC-rel pointer-to-GOT relocation.
      Type = MachO::ARM64_RELOC_POINTER_TO_GOT;
      IsPCRel = 1;
      MachO::any_relocation_info MRE;
      MRE.r_word0 = FixupOffset;
      MRE.r_word1 = (IsPCRel << 24) | (Log2Size << 25) | (Type << 28);
      Writer->addRelocation(A_Base, Fragment->getParent(), MRE);
      return;
    } else if (Target.getSpecifier() != AArch64::S_None) {
      // Otherwise, neither symbol can be modified.
      reportError(Fixup.getLoc(), "unsupported relocation of modified symbol");
      return;
    }

    // We don't support PCrel relocations of differences.
    if (IsPCRel) {
      reportError(Fixup.getLoc(), "unsupported pc-relative relocation of "
                                  "difference");
      return;
    }
```
**EN:** This comment block marks a new section and frames how the following implementation should be read.  
**CN:** 该注释块标记了新的代码章节，并为理解后续实现提供上下文。
### Lines 243-277: Function address
```cpp

    // AArch64 always uses external relocations. If there is no symbol to use as
    // a base address (a local symbol with no preceding non-local symbol),
    // error out.
    //
    // FIXME: We should probably just synthesize an external symbol and use
    // that.
    if (!A_Base) {
      reportError(Fixup.getLoc(),
                  "unsupported relocation of local symbol '" + A->getName() +
                      "'. Must have non-local symbol earlier in section.");
      return;
    }
    if (!B_Base) {
      reportError(Fixup.getLoc(),
                  "unsupported relocation of local symbol '" + B->getName() +
                      "'. Must have non-local symbol earlier in section.");
      return;
    }

    if (A_Base == B_Base && A_Base) {
      reportError(Fixup.getLoc(), "unsupported relocation with identical base");
      return;
    }

    Value +=
        (!A->getFragment() ? 0 : Writer->getSymbolAddress(*A)) -
        (!A_Base || !A_Base->getFragment() ? 0
                                           : Writer->getSymbolAddress(*A_Base));
    Value -=
        (!B->getFragment() ? 0 : Writer->getSymbolAddress(*B)) -
        (!B_Base || !B_Base->getFragment() ? 0
                                           : Writer->getSymbolAddress(*B_Base));

    Type = MachO::ARM64_RELOC_UNSIGNED;
```
**EN:** This block implements address, advancing the file's AArch64 backend logic flow with AArch64-specific decisions and data movement.  
**CN:** 该代码块实现 address，通过 AArch64 专用的决策与数据处理推进本文件的AArch64 后端逻辑流程。
### Lines 278-312: Core AArch64 backend logic
```cpp

    MachO::any_relocation_info MRE;
    MRE.r_word0 = FixupOffset;
    MRE.r_word1 = (IsPCRel << 24) | (Log2Size << 25) | (Type << 28);
    Writer->addRelocation(A_Base, Fragment->getParent(), MRE);

    RelSymbol = B_Base;
    Type = MachO::ARM64_RELOC_SUBTRACTOR;
  } else { // A + constant
    const MCSymbol *Symbol = Target.getAddSym();
    const MCSectionMachO &Section =
        static_cast<const MCSectionMachO &>(*Fragment->getParent());

    bool CanUseLocalRelocation =
        canUseLocalRelocation(Section, *Symbol, Log2Size);
    if (Symbol->isTemporary() && (Value || !CanUseLocalRelocation)) {
      // Make sure that the symbol is actually in a section here. If it isn't,
      // emit an error and exit.
      if (!Symbol->isInSection()) {
        reportError(Fixup.getLoc(),
                    "unsupported relocation of local symbol '" +
                        Symbol->getName() +
                        "'. Must have non-local symbol earlier in section.");
        return;
      }
      const MCSection &Sec = Symbol->getSection();
      if (!MCAsmInfoDarwin::isSectionAtomizableBySymbols(Sec))
        Symbol->setUsedInReloc();
    }

    const MCSymbol *Base = Writer->getAtom(*Symbol);

    // If the symbol is a variable it can either be in a section and
    // we have a base or it is absolute and should have been expanded.
    assert(!Symbol->isVariable() || Base);
```
**EN:** This block continues the file's main AArch64 backend logic logic and connects local helpers with the wider AArch64 backend.  
**CN:** 该代码块延续本文件的AArch64 后端逻辑主线逻辑，并把本地辅助实现连接到更广泛的 AArch64 后端中。
### Lines 313-344: Documented code section
```cpp

    // Relocations inside debug sections always use local relocations when
    // possible. This seems to be done because the debugger doesn't fully
    // understand relocation entries and expects to find values that
    // have already been fixed up.
    if (Symbol->isInSection()) {
      if (Section.hasAttribute(MachO::S_ATTR_DEBUG))
        Base = nullptr;
    }

    // AArch64 uses external relocations as much as possible. For debug
    // sections, and for pointer-sized relocations (.quad), we allow section
    // relocations.  It's code sections that run into trouble.
    if (Base) {
      RelSymbol = Base;

      // Add the local offset, if needed.
      if (Base != Symbol)
        Value += Asm.getSymbolOffset(*Symbol) - Asm.getSymbolOffset(*Base);
    } else if (Symbol->isInSection()) {
      if (!CanUseLocalRelocation) {
        reportError(Fixup.getLoc(),
                    "unsupported relocation of local symbol '" +
                        Symbol->getName() +
                        "'. Must have non-local symbol earlier in section.");
        return;
      }
      // Adjust the relocation to be section-relative.
      // The index is the section ordinal (1-based).
      const MCSection &Sec = Symbol->getSection();
      Index = Sec.getOrdinal() + 1;
      Value += Writer->getSymbolAddress(*Symbol);
```
**EN:** This comment block marks a new section and frames how the following implementation should be read.  
**CN:** 该注释块标记了新的代码章节，并为理解后续实现提供上下文。
### Lines 345-378: Core AArch64 backend logic
```cpp

      if (IsPCRel)
        Value -= Writer->getFragmentAddress(Asm, Fragment) + Fixup.getOffset() +
                 (1ULL << Log2Size);
    } else {
      llvm_unreachable(
          "This constant variable should have been expanded during evaluation");
    }
  }

  // If the relocation kind is Branch26, Page21, or Pageoff12, any addend
  // is represented via an Addend relocation, not encoded directly into
  // the instruction.
  if ((Type == MachO::ARM64_RELOC_BRANCH26 ||
       Type == MachO::ARM64_RELOC_PAGE21 ||
       Type == MachO::ARM64_RELOC_PAGEOFF12) &&
      Value) {
    if (!isInt<24>(Value)) {
      reportError(Fixup.getLoc(), "addend too big for relocation");
      return;
    }

    MachO::any_relocation_info MRE;
    MRE.r_word0 = FixupOffset;
    MRE.r_word1 =
        (Index << 0) | (IsPCRel << 24) | (Log2Size << 25) | (Type << 28);
    Writer->addRelocation(RelSymbol, Fragment->getParent(), MRE);

    // Now set up the Addend relocation.
    Type = MachO::ARM64_RELOC_ADDEND;
    Index = Value;
    RelSymbol = nullptr;
    IsPCRel = 0;
    Log2Size = 2;
```
**EN:** This block continues the file's main AArch64 backend logic logic and connects local helpers with the wider AArch64 backend.  
**CN:** 该代码块延续本文件的AArch64 后端逻辑主线逻辑，并把本地辅助实现连接到更广泛的 AArch64 后端中。
### Lines 379-413: Documented code section
```cpp

    // Put zero into the instruction itself. The addend is in the relocation.
    Value = 0;
  }

  if (Target.getSpecifier() == AArch64::S_AUTH ||
      Target.getSpecifier() == AArch64::S_AUTHADDR) {
    auto *Expr = cast<AArch64AuthMCExpr>(Fixup.getValue());

    assert(Type == MachO::ARM64_RELOC_UNSIGNED);

    if (IsPCRel) {
      reportError(Fixup.getLoc(), "invalid PC relative auth relocation");
      return;
    }

    if (Log2Size != 3) {
      reportError(Fixup.getLoc(),
                  "invalid auth relocation size, must be 8 bytes");
      return;
    }

    if (Target.getSubSym()) {
      reportError(Fixup.getLoc(),
                  "invalid auth relocation, can't reference two symbols");
      return;
    }

    uint16_t Discriminator = Expr->getDiscriminator();
    AArch64PACKey::ID Key = Expr->getKey();

    if (!isInt<32>(Value)) {
      reportError(Fixup.getLoc(), "addend too big for relocation");
      return;
    }
```
**EN:** This comment block marks a new section and frames how the following implementation should be read.  
**CN:** 该注释块标记了新的代码章节，并为理解后续实现提供上下文。
### Lines 414-437: Core AArch64 backend logic
```cpp

    Type = MachO::ARM64_RELOC_AUTHENTICATED_POINTER;
    Value = (uint32_t(Value)) | (uint64_t(Discriminator) << 32) |
            (uint64_t(Expr->hasAddressDiversity()) << 48) |
            (uint64_t(Key) << 49) | (1ULL << 63);
  }

  // If there's any addend left to handle, encode it in the instruction.
  FixedValue = Value;

  // struct relocation_info (8 bytes)
  MachO::any_relocation_info MRE;
  MRE.r_word0 = FixupOffset;
  MRE.r_word1 =
      (Index << 0) | (IsPCRel << 24) | (Log2Size << 25) | (Type << 28);
  Writer->addRelocation(RelSymbol, Fragment->getParent(), MRE);
}

std::unique_ptr<MCObjectTargetWriter>
llvm::createAArch64MachObjectWriter(uint32_t CPUType, uint32_t CPUSubtype,
                                    bool IsILP32) {
  return std::make_unique<AArch64MachObjectWriter>(CPUType, CPUSubtype,
                                                   IsILP32);
}
```
**EN:** This block continues the file's main AArch64 backend logic logic and connects local helpers with the wider AArch64 backend.  
**CN:** 该代码块延续本文件的AArch64 后端逻辑主线逻辑，并把本地辅助实现连接到更广泛的 AArch64 后端中。
## Key Concepts / 关键概念
- **EN:** AArch64 target backend structure **CN:** AArch64 目标后端结构
## Dependencies / 依赖关系
- **EN:** Target-local includes: MCTargetDesc/AArch64FixupKinds.h, MCTargetDesc/AArch64MCAsmInfo.h, MCTargetDesc/AArch64MCTargetDesc.h **CN:** 目标本地依赖：MCTargetDesc/AArch64FixupKinds.h, MCTargetDesc/AArch64MCAsmInfo.h, MCTargetDesc/AArch64MCTargetDesc.h
- **EN:** Core LLVM interfaces: llvm/ADT/Twine.h, llvm/BinaryFormat/MachO.h, llvm/MC/MCAsmInfo.h, llvm/MC/MCAsmInfoDarwin.h, llvm/MC/MCAssembler.h, llvm/MC/MCContext.h, llvm/MC/MCExpr.h, llvm/MC/MCFixup.h, llvm/MC/MCMachObjectWriter.h, llvm/MC/MCSection.h ... **CN:** 核心 LLVM 接口：llvm/ADT/Twine.h, llvm/BinaryFormat/MachO.h, llvm/MC/MCAsmInfo.h, llvm/MC/MCAsmInfoDarwin.h, llvm/MC/MCAssembler.h, llvm/MC/MCContext.h, llvm/MC/MCExpr.h, llvm/MC/MCFixup.h, llvm/MC/MCMachObjectWriter.h, llvm/MC/MCSection.h ...
- **EN:** Standard-library support: cassert, cstdint **CN:** 标准库支持：cassert, cstdint
- **EN:** Closely connected with neighboring AArch64 backend components responsible for AArch64 backend logic. **CN:** 与周边负责AArch64 后端逻辑的 AArch64 后端组件紧密协作。
