# MCXCOFFStreamer.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/MC/MCXCOFFStreamer.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: This file assembles .s files and emits XCOFF .o object files.
  - **CN**: 实现该文件在 LLVM 中对应的子系统逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- lib/MC/MCXCOFFStreamer.cpp - XCOFF Object Output -------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及文件级说明。

### Lines 8-21
```cpp
//
// This file assembles .s files and emits XCOFF .o object files.
//
//===----------------------------------------------------------------------===//

#include "llvm/MC/MCXCOFFStreamer.h"
#include "llvm/BinaryFormat/XCOFF.h"
#include "llvm/MC/MCAsmBackend.h"
#include "llvm/MC/MCAssembler.h"
#include "llvm/MC/MCCodeEmitter.h"
#include "llvm/MC/MCDirectives.h"
#include "llvm/MC/MCObjectWriter.h"
#include "llvm/MC/MCSectionXCOFF.h"
#include "llvm/MC/MCSymbolXCOFF.h"
```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/MC/MCXCOFFStreamer.h`, `llvm/BinaryFormat/XCOFF.h`, `llvm/MC/MCAsmBackend.h`, `llvm/MC/MCAssembler.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/MC/MCXCOFFStreamer.h`, `llvm/BinaryFormat/XCOFF.h`, `llvm/MC/MCAsmBackend.h`, `llvm/MC/MCAssembler.h`。

### Lines 22-33
```cpp
#include "llvm/MC/MCXCOFFObjectWriter.h"
#include "llvm/MC/TargetRegistry.h"

using namespace llvm;

MCXCOFFStreamer::MCXCOFFStreamer(MCContext &Context,
                                 std::unique_ptr<MCAsmBackend> MAB,
                                 std::unique_ptr<MCObjectWriter> OW,
                                 std::unique_ptr<MCCodeEmitter> Emitter)
    : MCObjectStreamer(Context, std::move(MAB), std::move(OW),
                       std::move(Emitter)) {}

```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/MC/MCXCOFFObjectWriter.h`, `llvm/MC/TargetRegistry.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/MC/MCXCOFFObjectWriter.h`, `llvm/MC/TargetRegistry.h`。

### Lines 34-47
```cpp
XCOFFObjectWriter &MCXCOFFStreamer::getWriter() {
  return static_cast<XCOFFObjectWriter &>(getAssembler().getWriter());
}

void MCXCOFFStreamer::changeSection(MCSection *Section, uint32_t Subsection) {
  MCObjectStreamer::changeSection(Section, Subsection);
  auto *Sec = static_cast<const MCSectionXCOFF *>(Section);
  // We might miss calculating the symbols difference as absolute value before
  // adding fixups when symbol_A without the fragment set is the csect itself
  // and symbol_B is in it.
  // TODO: Currently we only set the fragment for XMC_PR csects and DWARF
  // sections because we don't have other cases that hit this problem yet.
  // if (IsDwarfSec || CsectProp->MappingClass == XCOFF::XMC_PR)
  //   QualName->setFragment(F);
```
- **EN**: Implements logic around `getWriter`, `getAssembler`, `changeSection`; this block handles relocation, fixup, or symbol-resolution work; drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `getWriter`, `getAssembler`, `changeSection` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 48-55
```cpp
  if (Sec->isDwarfSect() || Sec->getMappingClass() == XCOFF::XMC_PR) {
    MCSymbol *QualNameSymbol = Sec->getQualNameSymbol();
    // Only set the fragment the first time we're switching to the section.
    if (!QualNameSymbol->isInSection())
      QualNameSymbol->setFragment(CurFrag);
  }
}

```
- **EN**: Implements logic around `getQualNameSymbol`, `setFragment`; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state; uses `switch`-based dispatch over enums, opcodes, or kinds.
- **CN**: 围绕 `getQualNameSymbol`, `setFragment` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，使用 `switch` 对枚举、opcode 或 kind 进行分派。

### Lines 56-65
```cpp
bool MCXCOFFStreamer::emitSymbolAttribute(MCSymbol *Sym,
                                          MCSymbolAttr Attribute) {
  auto *Symbol = static_cast<MCSymbolXCOFF *>(Sym);
  getAssembler().registerSymbol(*Symbol);

  switch (Attribute) {
  // XCOFF doesn't support the cold feature.
  case MCSA_Cold:
    return false;

```
- **EN**: Implements logic around `emitSymbolAttribute`, `getAssembler`; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `emitSymbolAttribute`, `getAssembler` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 66-79
```cpp
  case MCSA_Global:
  case MCSA_Extern:
    Symbol->setStorageClass(XCOFF::C_EXT);
    Symbol->setExternal(true);
    break;
  case MCSA_LGlobal:
    Symbol->setStorageClass(XCOFF::C_HIDEXT);
    Symbol->setExternal(true);
    break;
  case llvm::MCSA_Weak:
    Symbol->setStorageClass(XCOFF::C_WEAKEXT);
    Symbol->setExternal(true);
    break;
  case llvm::MCSA_Hidden:
```
- **EN**: Implements logic around `setStorageClass`, `setExternal`; this block updates MC section or symbol state.
- **CN**: 围绕 `setStorageClass`, `setExternal` 实现具体逻辑；这一段更新 MC 节区或符号状态。

### Lines 80-93
```cpp
    Symbol->setVisibilityType(XCOFF::SYM_V_HIDDEN);
    break;
  case llvm::MCSA_Protected:
    Symbol->setVisibilityType(XCOFF::SYM_V_PROTECTED);
    break;
  case llvm::MCSA_Exported:
    Symbol->setVisibilityType(XCOFF::SYM_V_EXPORTED);
    break;
  default:
    report_fatal_error("Not implemented yet.");
  }
  return true;
}

```
- **EN**: Implements logic around `setVisibilityType`, `report_fatal_error`; this block updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `setVisibilityType`, `report_fatal_error` 实现具体逻辑；这一段更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 94-102
```cpp
void MCXCOFFStreamer::emitXCOFFSymbolLinkageWithVisibility(
    MCSymbol *Symbol, MCSymbolAttr Linkage, MCSymbolAttr Visibility) {

  emitSymbolAttribute(Symbol, Linkage);

  // When the caller passes `MCSA_Invalid` for the visibility, do not emit one.
  if (Visibility == MCSA_Invalid)
    return;

```
- **EN**: Implements logic around `emitXCOFFSymbolLinkageWithVisibility`, `emitSymbolAttribute`; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `emitXCOFFSymbolLinkageWithVisibility`, `emitSymbolAttribute` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 103-112
```cpp
  emitSymbolAttribute(Symbol, Visibility);
}

void MCXCOFFStreamer::emitXCOFFRefDirective(const MCSymbol *Symbol) {
  // Add a Fixup here to later record a relocation of type R_REF to prevent the
  // ref symbol from being garbage collected (by the binder).
  addFixup(MCSymbolRefExpr::create(Symbol, getContext()),
           XCOFF::RelocationType::R_REF);
}

```
- **EN**: Implements logic around `emitSymbolAttribute`, `emitXCOFFRefDirective`, `addFixup`; this block handles relocation, fixup, or symbol-resolution work; parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `emitSymbolAttribute`, `emitXCOFFRefDirective`, `addFixup` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，解析汇编语法或操作数，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 113-119
```cpp
void MCXCOFFStreamer::emitXCOFFRenameDirective(const MCSymbol *Name,
                                               StringRef Rename) {
  auto *Symbol = static_cast<const MCSymbolXCOFF *>(Name);
  if (!Symbol->hasRename())
    report_fatal_error("Only explicit .rename is supported for XCOFF.");
}

```
- **EN**: Implements logic around `emitXCOFFRenameDirective`, `report_fatal_error`; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `emitXCOFFRenameDirective`, `report_fatal_error` 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 120-128
```cpp
void MCXCOFFStreamer::emitXCOFFExceptDirective(const MCSymbol *Symbol,
                                               const MCSymbol *Trap,
                                               unsigned Lang, unsigned Reason,
                                               unsigned FunctionSize,
                                               bool hasDebug) {
  getWriter().addExceptionEntry(Symbol, Trap, Lang, Reason, FunctionSize,
                                hasDebug);
}

```
- **EN**: Implements logic around `emitXCOFFExceptDirective`, `getWriter`; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `emitXCOFFExceptDirective`, `getWriter` 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 129-139
```cpp
void MCXCOFFStreamer::emitXCOFFCInfoSym(StringRef Name, StringRef Metadata) {
  getWriter().addCInfoSymEntry(Name, Metadata);
}

void MCXCOFFStreamer::emitCommonSymbol(MCSymbol *Symbol, uint64_t Size,
                                       Align ByteAlignment) {
  auto &Sym = static_cast<MCSymbolXCOFF &>(*Symbol);
  getAssembler().registerSymbol(*Symbol);
  Sym.setExternal(Sym.getStorageClass() != XCOFF::C_HIDEXT);
  Symbol->setCommon(Size, ByteAlignment);

```
- **EN**: Implements logic around `emitXCOFFCInfoSym`, `getWriter`, `emitCommonSymbol`, `getAssembler`, and 2 more symbols; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `emitXCOFFCInfoSym`, `getWriter`, `emitCommonSymbol`, `getAssembler`, and 2 more symbols 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 140-148
```cpp
  // Default csect align is 4, but common symbols have explicit alignment values
  // and we should honor it.
  Sym.getRepresentedCsect()->setAlignment(ByteAlignment);

  // Emit the alignment and storage for the variable to the section.
  emitValueToAlignment(ByteAlignment);
  emitZeros(Size);
}

```
- **EN**: Implements logic around `getRepresentedCsect`, `emitValueToAlignment`, `emitZeros`; this block updates MC section or symbol state.
- **CN**: 围绕 `getRepresentedCsect`, `emitValueToAlignment`, `emitZeros` 实现具体逻辑；这一段更新 MC 节区或符号状态。

### Lines 149-154
```cpp
void MCXCOFFStreamer::emitXCOFFLocalCommonSymbol(MCSymbol *LabelSym,
                                                 uint64_t Size,
                                                 MCSymbol *CsectSym,
                                                 Align Alignment) {
  emitCommonSymbol(CsectSym, Size, Alignment);
}
```
- **EN**: Implements logic around `emitXCOFFLocalCommonSymbol`, `emitCommonSymbol`; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `emitXCOFFLocalCommonSymbol`, `emitCommonSymbol` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

## Key Concepts / 关键概念

- **Relocations and fixups / 重定位与 fixup**:
  - **EN**: Bridges symbolic references to concrete addresses or relocation records
  - **CN**: 把符号引用连接到具体地址或重定位记录
- **Directive handling / 伪指令处理**:
  - **EN**: Interprets assembler directives and maps them to streamer operations
  - **CN**: 解释汇编伪指令并把它们映射为 streamer 操作
- **Debug/unwind metadata / 调试与展开元数据**:
  - **EN**: Emits or manages metadata needed for debugging and stack unwinding
  - **CN**: 输出或管理调试与栈展开所需的元数据
- **Object layout / 目标文件布局**:
  - **EN**: Organizes fragments into sections and computes final offsets
  - **CN**: 把片段组织到节区中并计算最终偏移

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `llvm/MC/MCXCOFFStreamer.h`, `llvm/BinaryFormat/XCOFF.h`, `llvm/MC/MCAsmBackend.h`, `llvm/MC/MCAssembler.h`, `llvm/MC/MCCodeEmitter.h`, `llvm/MC/MCDirectives.h`, `llvm/MC/MCObjectWriter.h`, `llvm/MC/MCSectionXCOFF.h`, `llvm/MC/MCSymbolXCOFF.h`, `llvm/MC/MCXCOFFObjectWriter.h`, `llvm/MC/TargetRegistry.h`
- **LLVM subsystems / LLVM 子系统**: MC, BinaryFormat
