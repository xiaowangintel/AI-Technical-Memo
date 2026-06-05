# X86ELFObjectWriter.cpp — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/X86/MCTargetDesc/X86ELFObjectWriter.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: Implements ELF object writing for the X86 MC target description layer. / 实现X86 MC 目标描述层中的ELF 目标文件写出。

## Line-by-Line Analysis / 逐行分析
### Lines 1-50: File header, licensing, and opening context / 文件头、许可证与开场上下文
```cpp
//===-- X86ELFObjectWriter.cpp - X86 ELF Writer ---------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "MCTargetDesc/X86FixupKinds.h"
#include "MCTargetDesc/X86MCAsmInfo.h"
#include "MCTargetDesc/X86MCTargetDesc.h"
#include "llvm/BinaryFormat/ELF.h"
#include "llvm/MC/MCAsmInfo.h"
#include "llvm/MC/MCContext.h"
#include "llvm/MC/MCELFObjectWriter.h"
#include "llvm/MC/MCExpr.h"
#include "llvm/MC/MCFixup.h"
#include "llvm/MC/MCObjectWriter.h"
#include "llvm/MC/MCValue.h"
#include "llvm/Support/ErrorHandling.h"
#include <cassert>
#include <cstdint>

using namespace llvm;

namespace {
enum X86_32RelType { RT32_NONE, RT32_32, RT32_16, RT32_8 };
enum X86_64RelType { RT64_NONE, RT64_64, RT64_32, RT64_32S, RT64_16, RT64_8 };

class X86ELFObjectWriter : public MCELFObjectTargetWriter {
public:
  X86ELFObjectWriter(bool IsELF64, uint8_t OSABI, uint16_t EMachine);
  ~X86ELFObjectWriter() override = default;

protected:
  unsigned getRelocType(const MCFixup &, const MCValue &,
                        bool IsPCRel) const override;
  bool needsRelocateWithSymbol(const MCValue &, unsigned Type) const override;

  void checkIs32(SMLoc Loc, X86_64RelType Type) const;
  void checkIs64(SMLoc Loc, X86_64RelType Type) const;
  unsigned getRelocType32(SMLoc Loc, X86::Specifier Specifier,
                          X86_32RelType Type, bool IsPCRel,
                          MCFixupKind Kind) const;
  unsigned getRelocType64(SMLoc Loc, X86::Specifier Specifier,
                          X86_64RelType Type, bool IsPCRel,
                          MCFixupKind Kind) const;
};

} // end anonymous namespace
```
**EN:** This section implements file header, licensing, and opening context for the X86 MC target description layer. Key symbols include X86_32RelType, X86_64RelType, X86ELFObjectWriter. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 MC 目标描述层中的文件头、许可证与开场上下文。关键符号包括 X86_32RelType, X86_64RelType, X86ELFObjectWriter。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 51-100: Function implementation and target-specific behavior / 函数实现与目标特定行为
```cpp

X86ELFObjectWriter::X86ELFObjectWriter(bool IsELF64, uint8_t OSABI,
                                       uint16_t EMachine)
    : MCELFObjectTargetWriter(IsELF64, OSABI, EMachine,
                              // Only i386 and IAMCU use Rel instead of RelA.
                              /*HasRelocationAddend*/
                              (EMachine != ELF::EM_386) &&
                                  (EMachine != ELF::EM_IAMCU)) {}

static X86_64RelType getType64(MCFixupKind Kind, X86::Specifier &Specifier,
                               bool &IsPCRel) {
  switch (unsigned(Kind)) {
  default:
    llvm_unreachable("Unimplemented");
  case FK_NONE:
    return RT64_NONE;
  case FK_Data_8:
    return RT64_64;
  case X86::reloc_signed_4byte:
  case X86::reloc_signed_4byte_relax:
    if (Specifier == X86::S_None && !IsPCRel)
      return RT64_32S;
    return RT64_32;
  case X86::reloc_global_offset_table:
    Specifier = X86::S_GOT;
    IsPCRel = true;
    return RT64_32;
  case FK_Data_4:
  case X86::reloc_riprel_4byte:
  case X86::reloc_riprel_4byte_relax:
  case X86::reloc_riprel_4byte_relax_rex:
  case X86::reloc_riprel_4byte_relax_rex2:
  case X86::reloc_riprel_4byte_movq_load:
  case X86::reloc_riprel_4byte_movq_load_rex2:
  case X86::reloc_riprel_4byte_relax_evex:
    return RT64_32;
  case X86::reloc_branch_4byte_pcrel:
    Specifier = X86::S_PLT;
    return RT64_32;
  case FK_Data_2:
    return RT64_16;
  case FK_Data_1:
    return RT64_8;
  }
}

void X86ELFObjectWriter::checkIs32(SMLoc Loc, X86_64RelType Type) const {
  if (Type != RT64_32)
    reportError(Loc, "32 bit reloc applied to a field with a different size");
}
```
**EN:** This section implements function implementation and target-specific behavior for the X86 MC target description layer. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 MC 目标描述层中的函数实现与目标特定行为。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 101-150: Function implementation and target-specific behavior / 函数实现与目标特定行为
```cpp

void X86ELFObjectWriter::checkIs64(SMLoc Loc, X86_64RelType Type) const {
  if (Type != RT64_64)
    reportError(Loc, "64 bit reloc applied to a field with a different size");
}

unsigned X86ELFObjectWriter::getRelocType64(SMLoc Loc, X86::Specifier Specifier,
                                            X86_64RelType Type, bool IsPCRel,
                                            MCFixupKind Kind) const {
  switch (Specifier) {
  default:
    llvm_unreachable("Unimplemented");
  case X86::S_None:
  case X86::S_ABS8:
    switch (Type) {
    case RT64_NONE:
      if (Specifier == X86::S_None)
        return ELF::R_X86_64_NONE;
      llvm_unreachable("Unimplemented");
    case RT64_64:
      return IsPCRel ? ELF::R_X86_64_PC64 : ELF::R_X86_64_64;
    case RT64_32:
      return IsPCRel ? ELF::R_X86_64_PC32 : ELF::R_X86_64_32;
    case RT64_32S:
      return ELF::R_X86_64_32S;
    case RT64_16:
      return IsPCRel ? ELF::R_X86_64_PC16 : ELF::R_X86_64_16;
    case RT64_8:
      return IsPCRel ? ELF::R_X86_64_PC8 : ELF::R_X86_64_8;
    }
    llvm_unreachable("unexpected relocation type!");
  case X86::S_GOT:
    switch (Type) {
    case RT64_64:
      return IsPCRel ? ELF::R_X86_64_GOTPC64 : ELF::R_X86_64_GOT64;
    case RT64_32:
      return IsPCRel ? ELF::R_X86_64_GOTPC32 : ELF::R_X86_64_GOT32;
    case RT64_32S:
    case RT64_16:
    case RT64_8:
    case RT64_NONE:
      llvm_unreachable("Unimplemented");
    }
    llvm_unreachable("unexpected relocation type!");
  case X86::S_GOTOFF:
    assert(!IsPCRel);
    if (Type != RT64_64)
      reportError(Loc, "unsupported relocation type");
    return ELF::R_X86_64_GOTOFF64;
  case X86::S_TPOFF:
```
**EN:** This section implements function implementation and target-specific behavior for the X86 MC target description layer. Key symbols include X86ELFObjectWriter::checkIs64. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 MC 目标描述层中的函数实现与目标特定行为。关键符号包括 X86ELFObjectWriter::checkIs64。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 151-200: Control flow, matching, and data updates / 控制流、匹配与数据更新
```cpp
    assert(!IsPCRel);
    switch (Type) {
    case RT64_64:
      return ELF::R_X86_64_TPOFF64;
    case RT64_32:
      return ELF::R_X86_64_TPOFF32;
    case RT64_32S:
    case RT64_16:
    case RT64_8:
    case RT64_NONE:
      llvm_unreachable("Unimplemented");
    }
    llvm_unreachable("unexpected relocation type!");
  case X86::S_DTPOFF:
    assert(!IsPCRel);
    switch (Type) {
    case RT64_64:
      return ELF::R_X86_64_DTPOFF64;
    case RT64_32:
      return ELF::R_X86_64_DTPOFF32;
    case RT64_32S:
    case RT64_16:
    case RT64_8:
    case RT64_NONE:
      llvm_unreachable("Unimplemented");
    }
    llvm_unreachable("unexpected relocation type!");
  case X86::S_SIZE:
    assert(!IsPCRel);
    switch (Type) {
    case RT64_64:
      return ELF::R_X86_64_SIZE64;
    case RT64_32:
      return ELF::R_X86_64_SIZE32;
    case RT64_32S:
    case RT64_16:
    case RT64_8:
    case RT64_NONE:
      llvm_unreachable("Unimplemented");
    }
    llvm_unreachable("unexpected relocation type!");
  case X86::S_TLSCALL:
    return ELF::R_X86_64_TLSDESC_CALL;
  case X86::S_TLSDESC:
    return ((unsigned)Kind == X86::reloc_riprel_4byte_relax_rex2)
               ? ELF::R_X86_64_CODE_4_GOTPC32_TLSDESC
               : ELF::R_X86_64_GOTPC32_TLSDESC;
  case X86::S_TLSGD:
    checkIs32(Loc, Type);
    return ELF::R_X86_64_TLSGD;
```
**EN:** This section implements control flow, matching, and data updates for the X86 MC target description layer. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 MC 目标描述层中的控制流、匹配与数据更新。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 201-250: Function implementation and target-specific behavior / 函数实现与目标特定行为
```cpp
  case X86::S_GOTTPOFF:
    checkIs32(Loc, Type);
    if ((unsigned)Kind == X86::reloc_riprel_4byte_movq_load_rex2 ||
        (unsigned)Kind == X86::reloc_riprel_4byte_relax_rex2)
      return ELF::R_X86_64_CODE_4_GOTTPOFF;
    else if ((unsigned)Kind == X86::reloc_riprel_4byte_relax_evex)
      return ELF::R_X86_64_CODE_6_GOTTPOFF;
    return ELF::R_X86_64_GOTTPOFF;
  case X86::S_TLSLD:
    checkIs32(Loc, Type);
    return ELF::R_X86_64_TLSLD;
  case X86::S_PLT:
    checkIs32(Loc, Type);
    return ELF::R_X86_64_PLT32;
  case X86::S_GOTPCREL:
    checkIs32(Loc, Type);
    // Older versions of ld.bfd/ld.gold/lld
    // do not support GOTPCRELX/REX_GOTPCRELX/CODE_4_GOTPCRELX,
    // and we want to keep back-compatibility.
    if (!getContext().getTargetOptions().X86RelaxRelocations)
      return ELF::R_X86_64_GOTPCREL;
    switch (unsigned(Kind)) {
    default:
      return ELF::R_X86_64_GOTPCREL;
    case X86::reloc_riprel_4byte_relax:
      return ELF::R_X86_64_GOTPCRELX;
    case X86::reloc_riprel_4byte_relax_rex:
    case X86::reloc_riprel_4byte_movq_load:
      return ELF::R_X86_64_REX_GOTPCRELX;
    case X86::reloc_riprel_4byte_relax_rex2:
    case X86::reloc_riprel_4byte_movq_load_rex2:
      return ELF::R_X86_64_CODE_4_GOTPCRELX;
    }
    llvm_unreachable("unexpected relocation type!");
  case X86::S_GOTPCREL_NORELAX:
    checkIs32(Loc, Type);
    return ELF::R_X86_64_GOTPCREL;
  case X86::S_PLTOFF:
    checkIs64(Loc, Type);
    return ELF::R_X86_64_PLTOFF64;
  }
}

unsigned X86ELFObjectWriter::getRelocType32(SMLoc Loc, X86::Specifier Specifier,
                                            X86_32RelType Type, bool IsPCRel,
                                            MCFixupKind Kind) const {
  switch (Specifier) {
  default:
    llvm_unreachable("Unimplemented");
  case X86::S_None:
```
**EN:** This section implements function implementation and target-specific behavior for the X86 MC target description layer. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 MC 目标描述层中的函数实现与目标特定行为。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 251-300: Function implementation and target-specific behavior / 函数实现与目标特定行为
```cpp
  case X86::S_ABS8:
    switch (Type) {
    case RT32_NONE:
      if (Specifier == X86::S_None)
        return ELF::R_386_NONE;
      llvm_unreachable("Unimplemented");
    case RT32_32:
      return IsPCRel ? ELF::R_386_PC32 : ELF::R_386_32;
    case RT32_16:
      return IsPCRel ? ELF::R_386_PC16 : ELF::R_386_16;
    case RT32_8:
      return IsPCRel ? ELF::R_386_PC8 : ELF::R_386_8;
    }
    llvm_unreachable("unexpected relocation type!");
  case X86::S_GOT:
    if (Type != RT32_32)
      break;
    if (IsPCRel)
      return ELF::R_386_GOTPC;
    // Older versions of ld.bfd/ld.gold/lld do not support R_386_GOT32X and we
    // want to maintain compatibility.
    if (!getContext().getTargetOptions().X86RelaxRelocations)
      return ELF::R_386_GOT32;

    return Kind == X86::reloc_signed_4byte_relax ? ELF::R_386_GOT32X
                                                 : ELF::R_386_GOT32;
  case X86::S_GOTOFF:
    assert(!IsPCRel);
    if (Type != RT32_32)
      break;
    return ELF::R_386_GOTOFF;
  case X86::S_TLSCALL:
    return ELF::R_386_TLS_DESC_CALL;
  case X86::S_TLSDESC:
    return ELF::R_386_TLS_GOTDESC;
  case X86::S_TPOFF:
    if (Type != RT32_32)
      break;
    assert(!IsPCRel);
    return ELF::R_386_TLS_LE_32;
  case X86::S_DTPOFF:
    if (Type != RT32_32)
      break;
    assert(!IsPCRel);
    return ELF::R_386_TLS_LDO_32;
  case X86::S_TLSGD:
    if (Type != RT32_32)
      break;
    assert(!IsPCRel);
    return ELF::R_386_TLS_GD;
```
**EN:** This section implements function implementation and target-specific behavior for the X86 MC target description layer. Key symbols include switch. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 MC 目标描述层中的函数实现与目标特定行为。关键符号包括 switch。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 301-350: Function implementation and target-specific behavior / 函数实现与目标特定行为
```cpp
  case X86::S_GOTTPOFF:
    if (Type != RT32_32)
      break;
    assert(!IsPCRel);
    return ELF::R_386_TLS_IE_32;
  case X86::S_PLT:
    if (Type != RT32_32)
      break;
    return ELF::R_386_PLT32;
  case X86::S_INDNTPOFF:
    if (Type != RT32_32)
      break;
    assert(!IsPCRel);
    return ELF::R_386_TLS_IE;
  case X86::S_NTPOFF:
    if (Type != RT32_32)
      break;
    assert(!IsPCRel);
    return ELF::R_386_TLS_LE;
  case X86::S_GOTNTPOFF:
    if (Type != RT32_32)
      break;
    assert(!IsPCRel);
    return ELF::R_386_TLS_GOTIE;
  case X86::S_TLSLDM:
    if (Type != RT32_32)
      break;
    assert(!IsPCRel);
    return ELF::R_386_TLS_LDM;
  }
  reportError(Loc, "unsupported relocation type");
  return ELF::R_386_NONE;
}

unsigned X86ELFObjectWriter::getRelocType(const MCFixup &Fixup,
                                          const MCValue &Target,
                                          bool IsPCRel) const {
  MCFixupKind Kind = Fixup.getKind();
  auto Specifier = X86::Specifier(Target.getSpecifier());
  switch (Specifier) {
  case X86::S_GOTTPOFF:
  case X86::S_INDNTPOFF:
  case X86::S_NTPOFF:
  case X86::S_GOTNTPOFF:
  case X86::S_TLSCALL:
  case X86::S_TLSDESC:
  case X86::S_TLSGD:
  case X86::S_TLSLD:
  case X86::S_TLSLDM:
  case X86::S_TPOFF:
```
**EN:** This section implements function implementation and target-specific behavior for the X86 MC target description layer. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 MC 目标描述层中的函数实现与目标特定行为。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 351-400: Function implementation and target-specific behavior / 函数实现与目标特定行为
```cpp
  case X86::S_DTPOFF:
    if (auto *S = const_cast<MCSymbol *>(Target.getAddSym()))
      static_cast<MCSymbolELF *>(S)->setType(ELF::STT_TLS);
    break;
  default:
    break;
  }

  X86_64RelType Type = getType64(Kind, Specifier, IsPCRel);
  if (getEMachine() == ELF::EM_X86_64)
    return getRelocType64(Fixup.getLoc(), Specifier, Type, IsPCRel, Kind);

  assert((getEMachine() == ELF::EM_386 || getEMachine() == ELF::EM_IAMCU) &&
         "Unsupported ELF machine type.");

  X86_32RelType RelType = RT32_NONE;
  switch (Type) {
  case RT64_NONE:
    break;
  case RT64_64:
    reportError(Fixup.getLoc(), "unsupported relocation type");
    return ELF::R_386_NONE;
  case RT64_32:
  case RT64_32S:
    RelType = RT32_32;
    break;
  case RT64_16:
    RelType = RT32_16;
    break;
  case RT64_8:
    RelType = RT32_8;
    break;
  }
  return getRelocType32(Fixup.getLoc(), Specifier, RelType, IsPCRel, Kind);
}

bool X86ELFObjectWriter::needsRelocateWithSymbol(const MCValue &V,
                                                 unsigned Type) const {
  switch (V.getSpecifier()) {
  case X86::S_GOT:
  case X86::S_PLT:
  case X86::S_GOTPCREL:
  case X86::S_GOTPCREL_NORELAX:
    return true;
  default:
    return false;
  }
}

std::unique_ptr<MCObjectTargetWriter>
```
**EN:** This section implements function implementation and target-specific behavior for the X86 MC target description layer. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 MC 目标描述层中的函数实现与目标特定行为。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 401-403: Supporting logic and local data handling / 辅助逻辑与局部数据处理
```cpp
llvm::createX86ELFObjectWriter(bool IsELF64, uint8_t OSABI, uint16_t EMachine) {
  return std::make_unique<X86ELFObjectWriter>(IsELF64, OSABI, EMachine);
}
```
**EN:** This section implements supporting logic and local data handling for the X86 MC target description layer. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 MC 目标描述层中的辅助逻辑与局部数据处理。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

## Key Concepts / 关键概念
- Primary topic: ELF object writing. / 核心主题：ELF 目标文件写出。
- Subsystem: the X86 MC target description layer. / 所属子系统：X86 MC 目标描述层。
- Notable symbols: X86_32RelType, X86_64RelType, X86ELFObjectWriter. / 重要符号：X86_32RelType, X86_64RelType, X86ELFObjectWriter。
- The file emphasizes executable backend logic and target-specific decision making. / 该文件以可执行后端逻辑和目标特定决策为主。
- The implementation follows LLVM namespace conventions. / 实现遵循 LLVM 的命名空间约定。

## Dependencies / 依赖关系
- Direct includes: MCTargetDesc/X86FixupKinds.h, MCTargetDesc/X86MCAsmInfo.h, MCTargetDesc/X86MCTargetDesc.h, llvm/BinaryFormat/ELF.h, llvm/MC/MCAsmInfo.h, llvm/MC/MCContext.h, llvm/MC/MCELFObjectWriter.h, llvm/MC/MCExpr.h, llvm/MC/MCFixup.h, llvm/MC/MCObjectWriter.h. / 直接包含：MCTargetDesc/X86FixupKinds.h, MCTargetDesc/X86MCAsmInfo.h, MCTargetDesc/X86MCTargetDesc.h, llvm/BinaryFormat/ELF.h, llvm/MC/MCAsmInfo.h, llvm/MC/MCContext.h, llvm/MC/MCELFObjectWriter.h, llvm/MC/MCExpr.h, llvm/MC/MCFixup.h, llvm/MC/MCObjectWriter.h。
- Additional include dependencies: 4 more headers. / 额外包含依赖：还有 4 个头文件。
- Integrates with shared LLVM backend layers such as IR, CodeGen, MC, and X86 target support as needed. / 按需与 LLVM 的 IR、CodeGen、MC 和 X86 目标支持层集成。
