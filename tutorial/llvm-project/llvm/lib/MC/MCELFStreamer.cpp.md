# MCELFStreamer.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/MC/MCELFStreamer.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: This file assembles .s files and emits ELF .o object files.
  - **CN**: 实现该文件在 LLVM 中对应的子系统逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp
//===- lib/MC/MCELFStreamer.cpp - ELF Object Output -----------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file assembles .s files and emits ELF .o object files.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及文件级说明。

### Lines 12-29
```cpp

#include "llvm/MC/MCELFStreamer.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/BinaryFormat/ELF.h"
#include "llvm/MC/MCAsmBackend.h"
#include "llvm/MC/MCAsmInfo.h"
#include "llvm/MC/MCAssembler.h"
#include "llvm/MC/MCCodeEmitter.h"
#include "llvm/MC/MCContext.h"
#include "llvm/MC/MCDirectives.h"
#include "llvm/MC/MCELFObjectWriter.h"
#include "llvm/MC/MCExpr.h"
#include "llvm/MC/MCFixup.h"
#include "llvm/MC/MCLFI.h"
#include "llvm/MC/MCObjectFileInfo.h"
#include "llvm/MC/MCObjectWriter.h"
#include "llvm/MC/MCSection.h"
#include "llvm/MC/MCSectionELF.h"
```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/MC/MCELFStreamer.h`, `llvm/ADT/SmallVector.h`, `llvm/BinaryFormat/ELF.h`, `llvm/MC/MCAsmBackend.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/MC/MCELFStreamer.h`, `llvm/ADT/SmallVector.h`, `llvm/BinaryFormat/ELF.h`, `llvm/MC/MCAsmBackend.h`。

### Lines 30-39
```cpp
#include "llvm/MC/MCStreamer.h"
#include "llvm/MC/MCSymbol.h"
#include "llvm/MC/MCSymbolELF.h"
#include "llvm/MC/MCTargetOptions.h"
#include "llvm/MC/TargetRegistry.h"
#include "llvm/Support/ErrorHandling.h"
#include "llvm/Support/LEB128.h"
#include <cassert>
#include <cstdint>

```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/MC/MCStreamer.h`, `llvm/MC/MCSymbol.h`, `llvm/MC/MCSymbolELF.h`, `llvm/MC/MCTargetOptions.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/MC/MCStreamer.h`, `llvm/MC/MCSymbol.h`, `llvm/MC/MCSymbolELF.h`, `llvm/MC/MCTargetOptions.h`。

### Lines 40-48
```cpp
using namespace llvm;

MCELFStreamer::MCELFStreamer(MCContext &Context,
                             std::unique_ptr<MCAsmBackend> TAB,
                             std::unique_ptr<MCObjectWriter> OW,
                             std::unique_ptr<MCCodeEmitter> Emitter)
    : MCObjectStreamer(Context, std::move(TAB), std::move(OW),
                       std::move(Emitter)) {}

```
- **EN**: Introduces declarations for `llvm`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 49-59
```cpp
ELFObjectWriter &MCELFStreamer::getWriter() {
  return static_cast<ELFObjectWriter &>(getAssembler().getWriter());
}

void MCELFStreamer::initSections(const MCSubtargetInfo &STI) {
  MCContext &Ctx = getContext();
  switchSection(Ctx.getObjectFileInfo()->getTextSection());
  emitCodeAlignment(Align(Ctx.getObjectFileInfo()->getTextSectionAlignment()),
                    &STI);
}

```
- **EN**: Implements logic around `getWriter`, `getAssembler`, `initSections`, `getContext`, and 2 more symbols; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `getWriter`, `getAssembler`, `initSections`, `getContext`, and 2 more symbols 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 60-69
```cpp
void MCELFStreamer::emitLabel(MCSymbol *S, SMLoc Loc) {
  auto *Symbol = static_cast<MCSymbolELF *>(S);
  MCObjectStreamer::emitLabel(Symbol, Loc);

  const MCSectionELF &Section =
      static_cast<const MCSectionELF &>(*getCurrentSectionOnly());
  if (Section.getFlags() & ELF::SHF_TLS)
    Symbol->setType(ELF::STT_TLS);
}

```
- **EN**: Implements logic around `emitLabel`, `getCurrentSectionOnly`, `setType`; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `emitLabel`, `getCurrentSectionOnly`, `setType` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 70-80
```cpp
void MCELFStreamer::emitLabelAtPos(MCSymbol *S, SMLoc Loc, MCFragment &F,
                                   uint64_t Offset) {
  auto *Symbol = static_cast<MCSymbolELF *>(S);
  MCObjectStreamer::emitLabelAtPos(Symbol, Loc, F, Offset);

  const MCSectionELF &Section =
      static_cast<const MCSectionELF &>(*getCurrentSectionOnly());
  if (Section.getFlags() & ELF::SHF_TLS)
    Symbol->setType(ELF::STT_TLS);
}

```
- **EN**: Implements logic around `emitLabelAtPos`, `getCurrentSectionOnly`, `setType`; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `emitLabelAtPos`, `getCurrentSectionOnly`, `setType` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 81-89
```cpp
void MCELFStreamer::changeSection(MCSection *Section, uint32_t Subsection) {
  MCAssembler &Asm = getAssembler();
  auto *SectionELF = static_cast<const MCSectionELF *>(Section);
  const MCSymbol *Grp = SectionELF->getGroup();
  if (Grp)
    Asm.registerSymbol(*Grp);
  if (SectionELF->getFlags() & ELF::SHF_GNU_RETAIN)
    getWriter().markGnuAbi();

```
- **EN**: Implements logic around `changeSection`, `getAssembler`, `getGroup`, `registerSymbol`, and 1 more symbols; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `changeSection`, `getAssembler`, `getGroup`, `registerSymbol`, and 1 more symbols 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 90-107
```cpp
  MCObjectStreamer::changeSection(Section, Subsection);
  auto *Sym = static_cast<MCSymbolELF *>(Section->getBeginSymbol());
  Sym->setBinding(ELF::STB_LOCAL);
  Sym->setType(ELF::STT_SECTION);
}

void MCELFStreamer::emitWeakReference(MCSymbol *Alias, const MCSymbol *Target) {
  auto *A = static_cast<MCSymbolELF *>(Alias);
  if (A->isDefined()) {
    getContext().reportError(getStartTokLoc(), "symbol '" + A->getName() +
                                                   "' is already defined");
    return;
  }
  A->setVariableValue(MCSymbolRefExpr::create(Target, getContext()));
  A->setIsWeakref();
  getWriter().Weakrefs.push_back(A);
}

```
- **EN**: Implements logic around `changeSection`, `getBeginSymbol`, `setBinding`, `setType`, and 5 more symbols; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `changeSection`, `getBeginSymbol`, `setBinding`, `setType`, and 5 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 108-123
```cpp
// When GNU as encounters more than one .type declaration for an object it seems
// to use a mechanism similar to the one below to decide which type is actually
// used in the object file.  The greater of T1 and T2 is selected based on the
// following ordering:
//  STT_NOTYPE < STT_OBJECT < STT_FUNC < STT_GNU_IFUNC < STT_TLS < anything else
// If neither T1 < T2 nor T2 < T1 according to this ordering, use T2 (the user
// provided type).
static unsigned CombineSymbolTypes(unsigned T1, unsigned T2) {
  for (unsigned Type : {ELF::STT_NOTYPE, ELF::STT_OBJECT, ELF::STT_FUNC,
                        ELF::STT_GNU_IFUNC, ELF::STT_TLS}) {
    if (T1 == Type)
      return T2;
    if (T2 == Type)
      return T1;
  }

```
- **EN**: Implements logic around `CombineSymbolTypes`; this block updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `CombineSymbolTypes` 实现具体逻辑；这一段更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 124-134
```cpp
  return T2;
}

bool MCELFStreamer::emitSymbolAttribute(MCSymbol *S, MCSymbolAttr Attribute) {
  auto *Symbol = static_cast<MCSymbolELF *>(S);

  // Adding a symbol attribute always introduces the symbol, note that an
  // important side effect of calling registerSymbol here is to register
  // the symbol with the assembler.
  getAssembler().registerSymbol(*Symbol);

```
- **EN**: Implements logic around `emitSymbolAttribute`, `getAssembler`; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `emitSymbolAttribute`, `getAssembler` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 135-152
```cpp
  // The implementation of symbol attributes is designed to match 'as', but it
  // leaves much to desired. It doesn't really make sense to arbitrarily add and
  // remove flags, but 'as' allows this (in particular, see .desc).
  //
  // In the future it might be worth trying to make these operations more well
  // defined.
  switch (Attribute) {
  case MCSA_Cold:
  case MCSA_Extern:
  case MCSA_LazyReference:
  case MCSA_Reference:
  case MCSA_SymbolResolver:
  case MCSA_PrivateExtern:
  case MCSA_WeakDefinition:
  case MCSA_WeakDefAutoPrivate:
  case MCSA_Invalid:
  case MCSA_IndirectSymbol:
  case MCSA_Exported:
```
- **EN**: Implements subsystem-specific case analysis using a `switch` over kinds, opcodes, or states.
- **CN**: 通过对 kind、opcode 或状态执行 `switch` 分析来实现子系统相关逻辑。

### Lines 153-161
```cpp
  case MCSA_WeakAntiDep:
  case MCSA_OSLinkage:
  case MCSA_XPLinkage:
    return false;

  case MCSA_NoDeadStrip:
    // Ignore for now.
    break;

```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 162-178
```cpp
  case MCSA_ELF_TypeGnuUniqueObject:
    Symbol->setType(CombineSymbolTypes(Symbol->getType(), ELF::STT_OBJECT));
    Symbol->setBinding(ELF::STB_GNU_UNIQUE);
    getWriter().markGnuAbi();
    break;

  case MCSA_Global:
    // For `.weak x; .global x`, GNU as sets the binding to STB_WEAK while we
    // traditionally set the binding to STB_GLOBAL. This is error-prone, so we
    // error on such cases. Note, we also disallow changed binding from .local.
    if (Symbol->isBindingSet() && Symbol->getBinding() != ELF::STB_GLOBAL)
      getContext().reportError(getStartTokLoc(),
                               Symbol->getName() +
                                   " changed binding to STB_GLOBAL");
    Symbol->setBinding(ELF::STB_GLOBAL);
    break;

```
- **EN**: Implements logic around `setType`, `setBinding`, `getWriter`, `getContext`, and 1 more symbols; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `setType`, `setBinding`, `getWriter`, `getContext`, and 1 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 179-188
```cpp
  case MCSA_WeakReference:
  case MCSA_Weak:
    // For `.global x; .weak x`, both MC and GNU as set the binding to STB_WEAK.
    // We emit a warning for now but may switch to an error in the future.
    if (Symbol->isBindingSet() && Symbol->getBinding() != ELF::STB_WEAK)
      getContext().reportWarning(
          getStartTokLoc(), Symbol->getName() + " changed binding to STB_WEAK");
    Symbol->setBinding(ELF::STB_WEAK);
    break;

```
- **EN**: Implements logic around `getContext`, `getStartTokLoc`, `setBinding`; this block parses assembly syntax or operands; updates MC section or symbol state; uses `switch`-based dispatch over enums, opcodes, or kinds.
- **CN**: 围绕 `getContext`, `getStartTokLoc`, `setBinding` 实现具体逻辑；这一段解析汇编语法或操作数，更新 MC 节区或符号状态，使用 `switch` 对枚举、opcode 或 kind 进行分派。

### Lines 189-200
```cpp
  case MCSA_Local:
    if (Symbol->isBindingSet() && Symbol->getBinding() != ELF::STB_LOCAL)
      getContext().reportError(getStartTokLoc(),
                               Symbol->getName() +
                                   " changed binding to STB_LOCAL");
    Symbol->setBinding(ELF::STB_LOCAL);
    break;

  case MCSA_ELF_TypeFunction:
    Symbol->setType(CombineSymbolTypes(Symbol->getType(), ELF::STT_FUNC));
    break;

```
- **EN**: Implements logic around `getContext`, `getName`, `setBinding`, `setType`; this block parses assembly syntax or operands; updates MC section or symbol state.
- **CN**: 围绕 `getContext`, `getName`, `setBinding`, `setType` 实现具体逻辑；这一段解析汇编语法或操作数，更新 MC 节区或符号状态。

### Lines 201-209
```cpp
  case MCSA_ELF_TypeIndFunction:
    Symbol->setType(CombineSymbolTypes(Symbol->getType(), ELF::STT_GNU_IFUNC));
    getWriter().markGnuAbi();
    break;

  case MCSA_ELF_TypeObject:
    Symbol->setType(CombineSymbolTypes(Symbol->getType(), ELF::STT_OBJECT));
    break;

```
- **EN**: Implements logic around `setType`, `getWriter`; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `setType`, `getWriter` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 210-218
```cpp
  case MCSA_ELF_TypeTLS:
    Symbol->setType(CombineSymbolTypes(Symbol->getType(), ELF::STT_TLS));
    break;

  case MCSA_ELF_TypeCommon:
    // TODO: Emit these as a common symbol.
    Symbol->setType(CombineSymbolTypes(Symbol->getType(), ELF::STT_OBJECT));
    break;

```
- **EN**: Implements logic around `setType`; this block updates MC section or symbol state.
- **CN**: 围绕 `setType` 实现具体逻辑；这一段更新 MC 节区或符号状态。

### Lines 219-230
```cpp
  case MCSA_ELF_TypeNoType:
    Symbol->setType(CombineSymbolTypes(Symbol->getType(), ELF::STT_NOTYPE));
    break;

  case MCSA_Protected:
    Symbol->setVisibility(ELF::STV_PROTECTED);
    break;

  case MCSA_Memtag:
    Symbol->setMemtag(true);
    break;

```
- **EN**: Implements logic around `setType`, `setVisibility`, `setMemtag`; this block updates MC section or symbol state.
- **CN**: 围绕 `setType`, `setVisibility`, `setMemtag` 实现具体逻辑；这一段更新 MC 节区或符号状态。

### Lines 231-241
```cpp
  case MCSA_Hidden:
    Symbol->setVisibility(ELF::STV_HIDDEN);
    break;

  case MCSA_Internal:
    Symbol->setVisibility(ELF::STV_INTERNAL);
    break;

  case MCSA_AltEntry:
    llvm_unreachable("ELF doesn't support the .alt_entry attribute");

```
- **EN**: Implements logic around `setVisibility`, `llvm_unreachable`; this block updates MC section or symbol state.
- **CN**: 围绕 `setVisibility`, `llvm_unreachable` 实现具体逻辑；这一段更新 MC 节区或符号状态。

### Lines 242-253
```cpp
  case MCSA_LGlobal:
    llvm_unreachable("ELF doesn't support the .lglobl attribute");
  }

  return true;
}

void MCELFStreamer::emitCommonSymbol(MCSymbol *S, uint64_t Size,
                                     Align ByteAlignment) {
  auto *Symbol = static_cast<MCSymbolELF *>(S);
  getAssembler().registerSymbol(*Symbol);

```
- **EN**: Implements logic around `llvm_unreachable`, `emitCommonSymbol`, `getAssembler`; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `llvm_unreachable`, `emitCommonSymbol`, `getAssembler` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 254-264
```cpp
  if (!Symbol->isBindingSet())
    Symbol->setBinding(ELF::STB_GLOBAL);

  Symbol->setType(ELF::STT_OBJECT);

  if (Symbol->getBinding() == ELF::STB_LOCAL) {
    MCSection &Section = *getAssembler().getContext().getELFSection(
        ".bss", ELF::SHT_NOBITS, ELF::SHF_WRITE | ELF::SHF_ALLOC);
    MCSectionSubPair P = getCurrentSection();
    switchSection(&Section);

```
- **EN**: Implements logic around `setBinding`, `setType`, `getAssembler`, `getCurrentSection`, and 1 more symbols; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state; uses `switch`-based dispatch over enums, opcodes, or kinds.
- **CN**: 围绕 `setBinding`, `setType`, `getAssembler`, `getCurrentSection`, and 1 more symbols 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，使用 `switch` 对枚举、opcode 或 kind 进行分派。

### Lines 265-275
```cpp
    emitValueToAlignment(ByteAlignment, 0, 1, 0);
    emitLabel(Symbol);
    emitZeros(Size);

    switchSection(P.first, P.second);
  } else {
    if (Symbol->declareCommon(Size, ByteAlignment))
      report_fatal_error(Twine("Symbol: ") + Symbol->getName() +
                         " redeclared as different type");
  }

```
- **EN**: Implements logic around `emitValueToAlignment`, `emitLabel`, `emitZeros`, `switchSection`, and 1 more symbols; this block updates MC section or symbol state; uses `switch`-based dispatch over enums, opcodes, or kinds.
- **CN**: 围绕 `emitValueToAlignment`, `emitLabel`, `emitZeros`, `switchSection`, and 1 more symbols 实现具体逻辑；这一段更新 MC 节区或符号状态，使用 `switch` 对枚举、opcode 或 kind 进行分派。

### Lines 276-289
```cpp
  Symbol->setSize(MCConstantExpr::create(Size, getContext()));
}

void MCELFStreamer::emitELFSize(MCSymbol *Symbol, const MCExpr *Value) {
  static_cast<MCSymbolELF *>(Symbol)->setSize(Value);
}

void MCELFStreamer::emitELFSymverDirective(const MCSymbol *OriginalSym,
                                           StringRef Name,
                                           bool KeepOriginalSym) {
  getWriter().Symvers.push_back(ELFObjectWriter::Symver{
      getStartTokLoc(), OriginalSym, Name, KeepOriginalSym});
}

```
- **EN**: Implements logic around `setSize`, `emitELFSize`, `emitELFSymverDirective`, `getWriter`, and 1 more symbols; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `setSize`, `emitELFSize`, `emitELFSymverDirective`, `getWriter`, and 1 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 290-298
```cpp
void MCELFStreamer::emitLocalCommonSymbol(MCSymbol *S, uint64_t Size,
                                          Align ByteAlignment) {
  auto *Symbol = static_cast<MCSymbolELF *>(S);
  // FIXME: Should this be caught and done earlier?
  getAssembler().registerSymbol(*Symbol);
  Symbol->setBinding(ELF::STB_LOCAL);
  emitCommonSymbol(Symbol, Size, ByteAlignment);
}

```
- **EN**: Implements logic around `emitLocalCommonSymbol`, `getAssembler`, `setBinding`, `emitCommonSymbol`; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `emitLocalCommonSymbol`, `getAssembler`, `setBinding`, `emitCommonSymbol` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 299-316
```cpp
void MCELFStreamer::emitCGProfileEntry(const MCSymbolRefExpr *From,
                                       const MCSymbolRefExpr *To,
                                       uint64_t Count) {
  getWriter().getCGProfile().push_back({From, To, Count});
}

void MCELFStreamer::emitIdent(StringRef IdentString) {
  MCSection *Comment = getAssembler().getContext().getELFSection(
      ".comment", ELF::SHT_PROGBITS, ELF::SHF_MERGE | ELF::SHF_STRINGS, 1);
  pushSection();
  switchSection(Comment);
  if (!SeenIdent) {
    emitInt8(0);
    SeenIdent = true;
  }
  emitBytes(IdentString);
  emitInt8(0);
  popSection();
```
- **EN**: Implements logic around `emitCGProfileEntry`, `getWriter`, `emitIdent`, `getAssembler`, and 5 more symbols; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state; uses `switch`-based dispatch over enums, opcodes, or kinds.
- **CN**: 围绕 `emitCGProfileEntry`, `getWriter`, `emitIdent`, `getAssembler`, and 5 more symbols 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，使用 `switch` 对枚举、opcode 或 kind 进行分派。

### Lines 317-334
```cpp
}

void MCELFStreamer::finalizeCGProfileEntry(const MCSymbolRefExpr *Sym,
                                           uint64_t Offset,
                                           const MCSymbolRefExpr *&SRE) {
  const MCSymbol *S = &SRE->getSymbol();
  if (S->isTemporary()) {
    if (!S->isInSection()) {
      getContext().reportError(
          SRE->getLoc(), Twine("Reference to undefined temporary symbol ") +
                             "`" + S->getName() + "`");
      return;
    }
    S = S->getSection().getBeginSymbol();
    S->setUsedInReloc();
    SRE = MCSymbolRefExpr::create(S, getContext(), SRE->getLoc());
  }
  auto *O = MCBinaryExpr::createAdd(
```
- **EN**: Implements logic around `finalizeCGProfileEntry`, `getSymbol`, `getContext`, `getLoc`, and 5 more symbols; this block handles relocation, fixup, or symbol-resolution work; drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `finalizeCGProfileEntry`, `getSymbol`, `getContext`, `getLoc`, and 5 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 335-352
```cpp
      Sym, MCConstantExpr::create(Offset, getContext()), getContext());
  MCObjectStreamer::emitRelocDirective(*O, "BFD_RELOC_NONE", SRE);
}

void MCELFStreamer::finalizeCGProfile() {
  ELFObjectWriter &W = getWriter();
  if (W.getCGProfile().empty())
    return;
  MCSection *CGProfile = getAssembler().getContext().getELFSection(
      ".llvm.call-graph-profile", ELF::SHT_LLVM_CALL_GRAPH_PROFILE,
      ELF::SHF_EXCLUDE, /*sizeof(Elf_CGProfile_Impl<>)=*/8);
  pushSection();
  switchSection(CGProfile);
  uint64_t Offset = 0;
  auto *Sym =
      MCSymbolRefExpr::create(CGProfile->getBeginSymbol(), getContext());
  for (auto &E : W.getCGProfile()) {
    finalizeCGProfileEntry(Sym, Offset, E.From);
```
- **EN**: Implements logic around `create`, `emitRelocDirective`, `finalizeCGProfile`, `getWriter`, and 4 more symbols; this block handles relocation, fixup, or symbol-resolution work; parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; updates MC section or symbol state; uses `switch`-based dispatch over enums, opcodes, or kinds.
- **CN**: 围绕 `create`, `emitRelocDirective`, `finalizeCGProfile`, `getWriter`, and 4 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，解析汇编语法或操作数，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，使用 `switch` 对枚举、opcode 或 kind 进行分派。

### Lines 353-364
```cpp
    finalizeCGProfileEntry(Sym, Offset, E.To);
    emitIntValue(E.Count, sizeof(uint64_t));
    Offset += sizeof(uint64_t);
  }
  popSection();
}

void MCELFStreamer::finishImpl() {
  // Emit .note.GNU-stack, similar to AsmPrinter::doFinalization.
  MCContext &Ctx = getContext();
  auto *StackSec = Ctx.getAsmInfo().getStackSection(Ctx,
                                                    /*Exec=*/false);
```
- **EN**: Implements logic around `finalizeCGProfileEntry`, `emitIntValue`, `popSection`, `finishImpl`, and 2 more symbols; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `finalizeCGProfileEntry`, `emitIntValue`, `popSection`, `finishImpl`, and 2 more symbols 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 365-374
```cpp
  if (StackSec && Ctx.getTargetOptions().MCNoExecStack)
    switchSection(StackSec);

  // Emit the .gnu attributes section if any attributes have been added.
  if (!GNUAttributes.empty()) {
    MCSection *DummyAttributeSection = nullptr;
    createAttributesSection("gnu", ".gnu.attributes", ELF::SHT_GNU_ATTRIBUTES,
                            DummyAttributeSection, GNUAttributes);
  }

```
- **EN**: Implements logic around `switchSection`, `createAttributesSection`; this block updates MC section or symbol state; uses `switch`-based dispatch over enums, opcodes, or kinds.
- **CN**: 围绕 `switchSection`, `createAttributesSection` 实现具体逻辑；这一段更新 MC 节区或符号状态，使用 `switch` 对枚举、opcode 或 kind 进行分派。

### Lines 375-383
```cpp
  if (Ctx.getTargetTriple().isLFI())
    emitLFINoteSection(*this, Ctx);

  finalizeCGProfile();
  emitFrames();

  this->MCObjectStreamer::finishImpl();
}

```
- **EN**: Implements logic around `emitLFINoteSection`, `finalizeCGProfile`, `emitFrames`, `finishImpl`; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `emitLFINoteSection`, `finalizeCGProfile`, `emitFrames`, `finishImpl` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 384-394
```cpp
void MCELFStreamer::setAttributeItem(unsigned Attribute, unsigned Value,
                                     bool OverwriteExisting) {
  // Look for existing attribute item
  if (AttributeItem *Item = getAttributeItem(Attribute)) {
    if (!OverwriteExisting)
      return;
    Item->Type = AttributeItem::NumericAttribute;
    Item->IntValue = Value;
    return;
  }

```
- **EN**: Implements logic around `setAttributeItem`; this block drives emission, layout, or binary encoding behavior.
- **CN**: 围绕 `setAttributeItem` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为。

### Lines 395-411
```cpp
  // Create new attribute item
  AttributeItem Item = {AttributeItem::NumericAttribute, Attribute, Value,
                        std::string(StringRef(""))};
  Contents.push_back(Item);
}

void MCELFStreamer::setAttributeItem(unsigned Attribute, StringRef Value,
                                     bool OverwriteExisting) {
  // Look for existing attribute item
  if (AttributeItem *Item = getAttributeItem(Attribute)) {
    if (!OverwriteExisting)
      return;
    Item->Type = AttributeItem::TextAttribute;
    Item->StringValue = std::string(Value);
    return;
  }

```
- **EN**: Implements logic around `string`, `push_back`, `setAttributeItem`; this block drives emission, layout, or binary encoding behavior.
- **CN**: 围绕 `string`, `push_back`, `setAttributeItem` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为。

### Lines 412-429
```cpp
  // Create new attribute item
  AttributeItem Item = {AttributeItem::TextAttribute, Attribute, 0,
                        std::string(Value)};
  Contents.push_back(Item);
}

void MCELFStreamer::setAttributeItems(unsigned Attribute, unsigned IntValue,
                                      StringRef StringValue,
                                      bool OverwriteExisting) {
  // Look for existing attribute item
  if (AttributeItem *Item = getAttributeItem(Attribute)) {
    if (!OverwriteExisting)
      return;
    Item->Type = AttributeItem::NumericAndTextAttributes;
    Item->IntValue = IntValue;
    Item->StringValue = std::string(StringValue);
    return;
  }
```
- **EN**: Implements logic around `string`, `push_back`, `setAttributeItems`; this block drives emission, layout, or binary encoding behavior.
- **CN**: 围绕 `string`, `push_back`, `setAttributeItems` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为。

### Lines 430-444
```cpp

  // Create new attribute item
  AttributeItem Item = {AttributeItem::NumericAndTextAttributes, Attribute,
                        IntValue, std::string(StringValue)};
  Contents.push_back(Item);
}

MCELFStreamer::AttributeItem *
MCELFStreamer::getAttributeItem(unsigned Attribute) {
  for (AttributeItem &Item : Contents)
    if (Item.Tag == Attribute)
      return &Item;
  return nullptr;
}

```
- **EN**: Implements logic around `string`, `push_back`, `getAttributeItem`; this block drives emission, layout, or binary encoding behavior; returns subsystem-specific computed results.
- **CN**: 围绕 `string`, `push_back`, `getAttributeItem` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，返回子系统相关的计算结果。

### Lines 445-462
```cpp
size_t MCELFStreamer::calculateContentSize(
    SmallVector<AttributeItem, 64> &AttrsVec) const {
  size_t Result = 0;
  for (const AttributeItem &Item : AttrsVec) {
    switch (Item.Type) {
    case AttributeItem::HiddenAttribute:
      break;
    case AttributeItem::NumericAttribute:
      Result += getULEB128Size(Item.Tag);
      Result += getULEB128Size(Item.IntValue);
      break;
    case AttributeItem::TextAttribute:
      Result += getULEB128Size(Item.Tag);
      Result += Item.StringValue.size() + 1; // string + '\0'
      break;
    case AttributeItem::NumericAndTextAttributes:
      Result += getULEB128Size(Item.Tag);
      Result += getULEB128Size(Item.IntValue);
```
- **EN**: Implements logic around `calculateContentSize`, `getULEB128Size`, `size`; this block drives emission, layout, or binary encoding behavior; uses `switch`-based dispatch over enums, opcodes, or kinds.
- **CN**: 围绕 `calculateContentSize`, `getULEB128Size`, `size` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，使用 `switch` 对枚举、opcode 或 kind 进行分派。

### Lines 463-480
```cpp
      Result += Item.StringValue.size() + 1; // string + '\0';
      break;
    }
  }
  return Result;
}

void MCELFStreamer::createAttributesSection(
    StringRef Vendor, const Twine &Section, unsigned Type,
    MCSection *&AttributeSection, SmallVector<AttributeItem, 64> &AttrsVec) {
  // <format-version>
  // [ <section-length> "vendor-name"
  // [ <file-tag> <size> <attribute>*
  //   | <section-tag> <size> <section-number>* 0 <attribute>*
  //   | <symbol-tag> <size> <symbol-number>* 0 <attribute>*
  //   ]+
  // ]*

```
- **EN**: Implements logic around `size`, `createAttributesSection`; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `size`, `createAttributesSection` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 481-491
```cpp
  // Switch section to AttributeSection or get/create the section.
  if (AttributeSection) {
    switchSection(AttributeSection);
  } else {
    AttributeSection = getContext().getELFSection(Section, Type, 0);
    switchSection(AttributeSection);

    // Format version
    emitInt8(0x41);
  }

```
- **EN**: Implements logic around `switchSection`, `getContext`, `emitInt8`; this block updates MC section or symbol state; uses `switch`-based dispatch over enums, opcodes, or kinds.
- **CN**: 围绕 `switchSection`, `getContext`, `emitInt8` 实现具体逻辑；这一段更新 MC 节区或符号状态，使用 `switch` 对枚举、opcode 或 kind 进行分派。

### Lines 492-503
```cpp
  // Vendor size + Vendor name + '\0'
  const size_t VendorHeaderSize = 4 + Vendor.size() + 1;

  // Tag + Tag Size
  const size_t TagHeaderSize = 1 + 4;

  const size_t ContentsSize = calculateContentSize(AttrsVec);

  emitInt32(VendorHeaderSize + TagHeaderSize + ContentsSize);
  emitBytes(Vendor);
  emitInt8(0); // '\0'

```
- **EN**: Implements logic around `size`, `calculateContentSize`, `emitInt32`, `emitBytes`, and 1 more symbols.
- **CN**: 围绕 `size`, `calculateContentSize`, `emitInt32`, `emitBytes`, and 1 more symbols 实现具体逻辑。

### Lines 504-521
```cpp
  emitInt8(ARMBuildAttrs::File);
  emitInt32(TagHeaderSize + ContentsSize);

  // Size should have been accounted for already, now
  // emit each field as its type (ULEB or String)
  for (const AttributeItem &Item : AttrsVec) {
    emitULEB128IntValue(Item.Tag);
    switch (Item.Type) {
    default:
      llvm_unreachable("Invalid attribute type");
    case AttributeItem::NumericAttribute:
      emitULEB128IntValue(Item.IntValue);
      break;
    case AttributeItem::TextAttribute:
      emitBytes(Item.StringValue);
      emitInt8(0); // '\0'
      break;
    case AttributeItem::NumericAndTextAttributes:
```
- **EN**: Implements logic around `emitInt8`, `emitInt32`, `emitULEB128IntValue`, `llvm_unreachable`, and 1 more symbols; this block uses `switch`-based dispatch over enums, opcodes, or kinds.
- **CN**: 围绕 `emitInt8`, `emitInt32`, `emitULEB128IntValue`, `llvm_unreachable`, and 1 more symbols 实现具体逻辑；这一段使用 `switch` 对枚举、opcode 或 kind 进行分派。

### Lines 522-531
```cpp
      emitULEB128IntValue(Item.IntValue);
      emitBytes(Item.StringValue);
      emitInt8(0); // '\0'
      break;
    }
  }

  AttrsVec.clear();
}

```
- **EN**: Implements logic around `emitULEB128IntValue`, `emitBytes`, `emitInt8`, `clear`.
- **CN**: 围绕 `emitULEB128IntValue`, `emitBytes`, `emitInt8`, `clear` 实现具体逻辑。

### Lines 532-544
```cpp
void MCELFStreamer::createAttributesWithSubsection(
    MCSection *&AttributeSection, const Twine &Section, unsigned Type,
    SmallVector<AttributeSubSection, 64> &SubSectionVec) {
  // <format-version: 'A'>
  // [ <uint32: subsection-length> NTBS: vendor-name
  //   <bytes: vendor-data>
  // ]*
  // vendor-data expends to:
  // <uint8: optional> <uint8: parameter type> <attribute>*
  if (0 == SubSectionVec.size()) {
    return;
  }

```
- **EN**: Implements logic around `createAttributesWithSubsection`; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `createAttributesWithSubsection` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 545-555
```cpp
  // Switch section to AttributeSection or get/create the section.
  if (AttributeSection) {
    switchSection(AttributeSection);
  } else {
    AttributeSection = getContext().getELFSection(Section, Type, 0);
    switchSection(AttributeSection);

    // Format version
    emitInt8(0x41);
  }

```
- **EN**: Implements logic around `switchSection`, `getContext`, `emitInt8`; this block updates MC section or symbol state; uses `switch`-based dispatch over enums, opcodes, or kinds.
- **CN**: 围绕 `switchSection`, `getContext`, `emitInt8` 实现具体逻辑；这一段更新 MC 节区或符号状态，使用 `switch` 对枚举、opcode 或 kind 进行分派。

### Lines 556-568
```cpp
  for (AttributeSubSection &SubSection : SubSectionVec) {
    // subsection-length + vendor-name + '\0'
    const size_t VendorHeaderSize = 4 + SubSection.VendorName.size() + 1;
    // optional + parameter-type
    const size_t VendorParameters = 1 + 1;
    const size_t ContentsSize = calculateContentSize(SubSection.Content);

    emitInt32(VendorHeaderSize + VendorParameters + ContentsSize);
    emitBytes(SubSection.VendorName);
    emitInt8(0); // '\0'
    emitInt8(SubSection.IsOptional);
    emitInt8(SubSection.ParameterType);

```
- **EN**: Implements logic around `size`, `calculateContentSize`, `emitInt32`, `emitBytes`, and 1 more symbols; this block updates MC section or symbol state.
- **CN**: 围绕 `size`, `calculateContentSize`, `emitInt32`, `emitBytes`, and 1 more symbols 实现具体逻辑；这一段更新 MC 节区或符号状态。

### Lines 569-586
```cpp
    for (AttributeItem &Item : SubSection.Content) {
      emitULEB128IntValue(Item.Tag);
      switch (Item.Type) {
      default:
        assert(0 && "Invalid attribute type");
        break;
      case AttributeItem::NumericAttribute:
        emitULEB128IntValue(Item.IntValue);
        break;
      case AttributeItem::TextAttribute:
        emitBytes(Item.StringValue);
        emitInt8(0); // '\0'
        break;
      case AttributeItem::NumericAndTextAttributes:
        emitULEB128IntValue(Item.IntValue);
        emitBytes(Item.StringValue);
        emitInt8(0); // '\0'
        break;
```
- **EN**: Implements logic around `emitULEB128IntValue`, `assert`, `emitBytes`, `emitInt8`; this block updates MC section or symbol state; uses `switch`-based dispatch over enums, opcodes, or kinds.
- **CN**: 围绕 `emitULEB128IntValue`, `assert`, `emitBytes`, `emitInt8` 实现具体逻辑；这一段更新 MC 节区或符号状态，使用 `switch` 对枚举、opcode 或 kind 进行分派。

### Lines 587-600
```cpp
      }
    }
  }
  SubSectionVec.clear();
}

MCStreamer *llvm::createELFStreamer(MCContext &Context,
                                    std::unique_ptr<MCAsmBackend> &&MAB,
                                    std::unique_ptr<MCObjectWriter> &&OW,
                                    std::unique_ptr<MCCodeEmitter> &&CE) {
  MCELFStreamer *S =
      new MCELFStreamer(Context, std::move(MAB), std::move(OW), std::move(CE));
  return S;
}
```
- **EN**: Implements logic around `clear`, `createELFStreamer`, `MCELFStreamer`; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `clear`, `createELFStreamer`, `MCELFStreamer` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

## Key Concepts / 关键概念

- **Relocations and fixups / 重定位与 fixup**:
  - **EN**: Bridges symbolic references to concrete addresses or relocation records
  - **CN**: 把符号引用连接到具体地址或重定位记录
- **Directive handling / 伪指令处理**:
  - **EN**: Interprets assembler directives and maps them to streamer operations
  - **CN**: 解释汇编伪指令并把它们映射为 streamer 操作
- **Object layout / 目标文件布局**:
  - **EN**: Organizes fragments into sections and computes final offsets
  - **CN**: 把片段组织到节区中并计算最终偏移

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `llvm/MC/MCELFStreamer.h`, `llvm/ADT/SmallVector.h`, `llvm/BinaryFormat/ELF.h`, `llvm/MC/MCAsmBackend.h`, `llvm/MC/MCAsmInfo.h`, `llvm/MC/MCAssembler.h`, `llvm/MC/MCCodeEmitter.h`, `llvm/MC/MCContext.h`, `llvm/MC/MCDirectives.h`, `llvm/MC/MCELFObjectWriter.h`, `llvm/MC/MCExpr.h`, `llvm/MC/MCFixup.h` ... (+14 more)
- **LLVM subsystems / LLVM 子系统**: MC, Support, BinaryFormat
