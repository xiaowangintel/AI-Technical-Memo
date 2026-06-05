# XCOFFLinkGraphBuilder.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/ExecutionEngine/JITLink/XCOFFLinkGraphBuilder.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Implements JITLink graph-based linking, relocation handling, and format-specific link graph passes for just-in-time compiled code.
  - **CN**: 实现面向即时编译代码的 JITLink 图链接、重定位处理以及特定文件格式的链接图 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-9
```cpp
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// Generic XCOFF LinkGraph building code.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及文件级说明。

### Lines 10-25
```cpp

#include "XCOFFLinkGraphBuilder.h"
#include "llvm/ADT/STLExtras.h"
#include "llvm/BinaryFormat/XCOFF.h"
#include "llvm/ExecutionEngine/JITLink/JITLink.h"
#include "llvm/ExecutionEngine/JITLink/ppc64.h"
#include "llvm/ExecutionEngine/Orc/Shared/ExecutorAddress.h"
#include "llvm/ExecutionEngine/Orc/Shared/MemoryFlags.h"
#include "llvm/Object/ObjectFile.h"
#include "llvm/Object/XCOFFObjectFile.h"
#include "llvm/Support/Debug.h"
#include "llvm/Support/Error.h"
#include "llvm/Support/Format.h"
#include "llvm/Support/raw_ostream.h"
#include <memory>

```
- **EN**: Pulls in the headers needed for this implementation, including `XCOFFLinkGraphBuilder.h`, `llvm/ADT/STLExtras.h`, `llvm/BinaryFormat/XCOFF.h`, `llvm/ExecutionEngine/JITLink/JITLink.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `XCOFFLinkGraphBuilder.h`, `llvm/ADT/STLExtras.h`, `llvm/BinaryFormat/XCOFF.h`, `llvm/ExecutionEngine/JITLink/JITLink.h`。

### Lines 26-42
```cpp
using namespace llvm;

#define DEBUG_TYPE "jitlink"

namespace llvm {
namespace jitlink {

XCOFFLinkGraphBuilder::XCOFFLinkGraphBuilder(
    const object::XCOFFObjectFile &Obj,
    std::shared_ptr<orc::SymbolStringPool> SSP, Triple TT,
    SubtargetFeatures Features,
    LinkGraph::GetEdgeKindNameFunction GetEdgeKindName)
    : Obj(Obj),
      G(std::make_unique<LinkGraph>(
          std::string(Obj.getFileName()), std::move(SSP), std::move(TT),
          std::move(Features), std::move(GetEdgeKindName))) {}

```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 43-60
```cpp
#ifndef NDEBUG
static llvm::StringRef getStorageClassString(XCOFF::StorageClass SC) {
  switch (SC) {
  case XCOFF::StorageClass::C_FILE:
    return "C_FILE (File name)";
  case XCOFF::StorageClass::C_BINCL:
    return "C_BINCL (Beginning of include file)";
  case XCOFF::StorageClass::C_EINCL:
    return "C_EINCL (Ending of include file)";
  case XCOFF::StorageClass::C_GSYM:
    return "C_GSYM (Global variable)";
  case XCOFF::StorageClass::C_STSYM:
    return "C_STSYM (Statically allocated symbol)";
  case XCOFF::StorageClass::C_BCOMM:
    return "C_BCOMM (Beginning of common block)";
  case XCOFF::StorageClass::C_ECOMM:
    return "C_ECOMM (End of common block)";
  case XCOFF::StorageClass::C_ENTRY:
```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 61-78
```cpp
    return "C_ENTRY (Alternate entry)";
  case XCOFF::StorageClass::C_BSTAT:
    return "C_BSTAT (Beginning of static block)";
  case XCOFF::StorageClass::C_ESTAT:
    return "C_ESTAT (End of static block)";
  case XCOFF::StorageClass::C_GTLS:
    return "C_GTLS (Global thread-local variable)";
  case XCOFF::StorageClass::C_STTLS:
    return "C_STTLS (Static thread-local variable)";
  case XCOFF::StorageClass::C_DWARF:
    return "C_DWARF (DWARF section symbol)";
  case XCOFF::StorageClass::C_LSYM:
    return "C_LSYM (Automatic variable allocated on stack)";
  case XCOFF::StorageClass::C_PSYM:
    return "C_PSYM (Argument to subroutine allocated on stack)";
  case XCOFF::StorageClass::C_RSYM:
    return "C_RSYM (Register variable)";
  case XCOFF::StorageClass::C_RPSYM:
```
- **EN**: Implements logic around `C_ENTRY`, `C_BSTAT`, `C_ESTAT`, `C_GTLS`, and 5 more symbols; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `C_ENTRY`, `C_BSTAT`, `C_ESTAT`, `C_GTLS`, and 5 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 79-96
```cpp
    return "C_RPSYM (Argument to function stored in register)";
  case XCOFF::StorageClass::C_ECOML:
    return "C_ECOML (Local member of common block)";
  case XCOFF::StorageClass::C_FUN:
    return "C_FUN (Function or procedure)";
  case XCOFF::StorageClass::C_EXT:
    return "C_EXT (External symbol)";
  case XCOFF::StorageClass::C_WEAKEXT:
    return "C_WEAKEXT (Weak external symbol)";
  case XCOFF::StorageClass::C_NULL:
    return "C_NULL";
  case XCOFF::StorageClass::C_STAT:
    return "C_STAT (Static)";
  case XCOFF::StorageClass::C_BLOCK:
    return "C_BLOCK (\".bb\" or \".eb\")";
  case XCOFF::StorageClass::C_FCN:
    return "C_FCN (\".bf\" or \".ef\")";
  case XCOFF::StorageClass::C_HIDEXT:
```
- **EN**: Implements logic around `C_RPSYM`, `C_ECOML`, `C_FUN`, `C_EXT`, and 4 more symbols; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `C_RPSYM`, `C_ECOML`, `C_FUN`, `C_EXT`, and 4 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 97-114
```cpp
    return "C_HIDEXT (Un-named external symbol)";
  case XCOFF::StorageClass::C_INFO:
    return "C_INFO (Comment string in .info section)";
  case XCOFF::StorageClass::C_DECL:
    return "C_DECL (Declaration of object)";
  case XCOFF::StorageClass::C_AUTO:
    return "C_AUTO (Automatic variable)";
  case XCOFF::StorageClass::C_REG:
    return "C_REG (Register variable)";
  case XCOFF::StorageClass::C_EXTDEF:
    return "C_EXTDEF (External definition)";
  case XCOFF::StorageClass::C_LABEL:
    return "C_LABEL (Label)";
  case XCOFF::StorageClass::C_ULABEL:
    return "C_ULABEL (Undefined label)";
  case XCOFF::StorageClass::C_MOS:
    return "C_MOS (Member of structure)";
  case XCOFF::StorageClass::C_ARG:
```
- **EN**: Implements logic around `C_HIDEXT`, `C_INFO`, `C_DECL`, `C_AUTO`, and 5 more symbols; this block returns subsystem-specific computed results.
- **CN**: 围绕 `C_HIDEXT`, `C_INFO`, `C_DECL`, `C_AUTO`, and 5 more symbols 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 115-132
```cpp
    return "C_ARG (Function argument)";
  case XCOFF::StorageClass::C_STRTAG:
    return "C_STRTAG (Structure tag)";
  case XCOFF::StorageClass::C_MOU:
    return "C_MOU (Member of union)";
  case XCOFF::StorageClass::C_UNTAG:
    return "C_UNTAG (Union tag)";
  case XCOFF::StorageClass::C_TPDEF:
    return "C_TPDEF (Type definition)";
  case XCOFF::StorageClass::C_USTATIC:
    return "C_USTATIC (Undefined static)";
  case XCOFF::StorageClass::C_ENTAG:
    return "C_ENTAG (Enumeration tag)";
  case XCOFF::StorageClass::C_MOE:
    return "C_MOE (Member of enumeration)";
  case XCOFF::StorageClass::C_REGPARM:
    return "C_REGPARM (Register parameter)";
  case XCOFF::StorageClass::C_FIELD:
```
- **EN**: Implements logic around `C_ARG`, `C_STRTAG`, `C_MOU`, `C_UNTAG`, and 5 more symbols; this block returns subsystem-specific computed results.
- **CN**: 围绕 `C_ARG`, `C_STRTAG`, `C_MOU`, `C_UNTAG`, and 5 more symbols 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 133-150
```cpp
    return "C_FIELD (Bit field)";
  case XCOFF::StorageClass::C_EOS:
    return "C_EOS (End of structure)";
  case XCOFF::StorageClass::C_LINE:
    return "C_LINE";
  case XCOFF::StorageClass::C_ALIAS:
    return "C_ALIAS (Duplicate tag)";
  case XCOFF::StorageClass::C_HIDDEN:
    return "C_HIDDEN (Special storage class for external)";
  case XCOFF::StorageClass::C_EFCN:
    return "C_EFCN (Physical end of function)";
  case XCOFF::StorageClass::C_TCSYM:
    return "C_TCSYM (Reserved)";
  }
  llvm_unreachable("Unknown XCOFF::StorageClass enum");
}
#endif

```
- **EN**: Introduces declarations for `for`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `for` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 151-160
```cpp
Error XCOFFLinkGraphBuilder::processSections() {
  LLVM_DEBUG(dbgs() << "  Creating graph sections...\n");

  UndefSection = &G->createSection("*UND*", orc::MemProt::None);

  for (object::SectionRef Section : Obj.sections()) {
    auto SectionName = Section.getName();
    if (!SectionName)
      return SectionName.takeError();

```
- **EN**: Implements logic around `processSections`, `createSection`, `getName`, `takeError`; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `processSections`, `createSection`, `getName`, `takeError` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 161-172
```cpp
    LLVM_DEBUG({
      dbgs() << "    section = " << *SectionName
             << ", idx = " << Section.getIndex()
             << ", size = " << format_hex_no_prefix(Section.getSize(), 8)
             << ", vma = " << format_hex(Section.getAddress(), 16) << "\n";
    });

    // We can skip debug (including dawrf) and pad sections
    if (Section.isDebugSection() || *SectionName == "pad")
      continue;
    LLVM_DEBUG(dbgs() << "        creating graph section\n");

```
- **EN**: Implements logic around `dbgs`, `getIndex`, `format_hex_no_prefix`, `format_hex`.
- **CN**: 围绕 `dbgs`, `getIndex`, `format_hex_no_prefix`, `format_hex` 实现具体逻辑。

### Lines 173-181
```cpp
    orc::MemProt Prot = orc::MemProt::Read;
    if (Section.isText())
      Prot |= orc::MemProt::Exec;
    if (Section.isData() || Section.isBSS())
      Prot |= orc::MemProt::Write;

    jitlink::Section *GraphSec = &G->createSection(*SectionName, Prot);
    // TODO: Check for no_alloc for certain sections

```
- **EN**: Implements logic around `createSection`; this block drives emission, layout, or binary encoding behavior.
- **CN**: 围绕 `createSection` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为。

### Lines 182-197
```cpp
    assert(!SectionTable.contains(Section.getIndex()) &&
           "Section with same index already exists");
    SectionTable[Section.getIndex()] = {GraphSec, Section};
  }

  return Error::success();
}

static std::optional<object::XCOFFSymbolRef>
getXCOFFSymbolContainingSymbolRef(const object::XCOFFObjectFile &Obj,
                                  const object::SymbolRef &Sym) {
  const object::XCOFFSymbolRef SymRef =
      Obj.toSymbolRef(Sym.getRawDataRefImpl());
  if (!SymRef.isCsectSymbol())
    return std::nullopt;

```
- **EN**: Implements logic around `assert`, `getIndex`, `success`, `getXCOFFSymbolContainingSymbolRef`, and 1 more symbols; this block returns subsystem-specific computed results.
- **CN**: 围绕 `assert`, `getIndex`, `success`, `getXCOFFSymbolContainingSymbolRef`, and 1 more symbols 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 198-208
```cpp
  Expected<object::XCOFFCsectAuxRef> CsectAuxEntOrErr =
      SymRef.getXCOFFCsectAuxRef();
  if (!CsectAuxEntOrErr || !CsectAuxEntOrErr.get().isLabel())
    return std::nullopt;
  uint32_t Idx =
      static_cast<uint32_t>(CsectAuxEntOrErr.get().getSectionOrLength());
  object::DataRefImpl DRI;
  DRI.p = Obj.getSymbolByIndex(Idx);
  return object::XCOFFSymbolRef(DRI, &Obj);
}

```
- **EN**: Implements logic around `getXCOFFCsectAuxRef`, `static_cast<uint32_t>`, `getSymbolByIndex`, `XCOFFSymbolRef`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `getXCOFFCsectAuxRef`, `static_cast<uint32_t>`, `getSymbolByIndex`, `XCOFFSymbolRef` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 209-226
```cpp
#ifndef NDEBUG
static void printSymbolEntry(raw_ostream &OS,
                             const object::XCOFFObjectFile &Obj,
                             const object::XCOFFSymbolRef &Sym) {
  OS << "    " << format_hex(cantFail(Sym.getAddress()), 16);
  OS << " " << left_justify(cantFail(Sym.getName()), 10);
  if (Sym.isCsectSymbol()) {
    auto CsectAuxEntry = cantFail(Sym.getXCOFFCsectAuxRef());
    if (!CsectAuxEntry.isLabel()) {
      std::string MCStr =
          "[" +
          XCOFF::getMappingClassString(CsectAuxEntry.getStorageMappingClass())
              .str() +
          "]";
      OS << left_justify(MCStr, 3);
    }
  }
  OS << " " << format_hex(Sym.getSize(), 8);
```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 227-239
```cpp
  OS << " " << Sym.getSectionNumber();
  OS << " " << getStorageClassString(Sym.getStorageClass());
  OS << " (idx: " << Obj.getSymbolIndex(Sym.getRawDataRefImpl().p) << ")";
  if (Sym.isCsectSymbol()) {
    if (auto ParentSym = getXCOFFSymbolContainingSymbolRef(Obj, Sym)) {
      OS << " (csect idx: "
         << Obj.getSymbolIndex(ParentSym->getRawDataRefImpl().p) << ")";
    }
  }
  OS << "\n";
}
#endif

```
- **EN**: Implements logic around `getSectionNumber`, `getStorageClassString`, `getSymbolIndex`.
- **CN**: 围绕 `getSectionNumber`, `getStorageClassString`, `getSymbolIndex` 实现具体逻辑。

### Lines 240-250
```cpp
Error XCOFFLinkGraphBuilder::processCsectsAndSymbols() {
  LLVM_DEBUG(dbgs() << "  Creating graph blocks and symbols...\n");

  for ([[maybe_unused]] auto [K, V] : SectionTable) {
    LLVM_DEBUG(dbgs() << "    section entry(idx: " << K
                      << " section: " << V.Section->getName() << ")\n");
  }

  for (object::XCOFFSymbolRef Symbol : Obj.symbols()) {
    LLVM_DEBUG({ printSymbolEntry(dbgs(), Obj, Symbol); });

```
- **EN**: Implements logic around `processCsectsAndSymbols`, `getName`; this block manipulates JITLink graph structures or link-time passes.
- **CN**: 围绕 `processCsectsAndSymbols`, `getName` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass。

### Lines 251-263
```cpp
    auto Flags = Symbol.getFlags();
    if (!Flags)
      return Flags.takeError();

    bool External = *Flags & object::SymbolRef::SF_Undefined;
    bool Weak = *Flags & object::SymbolRef::SF_Weak;
    bool Global = *Flags & object::SymbolRef::SF_Global;

    auto SymbolIndex = Obj.getSymbolIndex(Symbol.getEntryAddress());
    auto SymbolName = Symbol.getName();
    if (!SymbolName)
      return SymbolName.takeError();

```
- **EN**: Implements logic around `getFlags`, `takeError`, `getSymbolIndex`, `getName`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `getFlags`, `takeError`, `getSymbolIndex`, `getName` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 264-275
```cpp
    if (External) {
      LLVM_DEBUG(dbgs() << "      created external symbol\n");
      SymbolIndexTable[SymbolIndex] =
          &G->addExternalSymbol(*SymbolName, Symbol.getSize(), Weak);
      continue;
    }

    if (!Symbol.isCsectSymbol()) {
      LLVM_DEBUG(dbgs() << "      skipped: not a csect symbol\n");
      continue;
    }

```
- **EN**: Implements logic around `addExternalSymbol`.
- **CN**: 围绕 `addExternalSymbol` 实现具体逻辑。

### Lines 276-287
```cpp
    auto ParentSym = getXCOFFSymbolContainingSymbolRef(Obj, Symbol);
    object::XCOFFSymbolRef CsectSymbol = ParentSym ? *ParentSym : Symbol;

    auto CsectSymbolIndex = Obj.getSymbolIndex(CsectSymbol.getEntryAddress());
    auto ParentSectionNumber = CsectSymbol.getSectionNumber();

    bool IsUndefinedSection = !SectionTable.contains(ParentSectionNumber);
    Section *ParentSection = !IsUndefinedSection
                                 ? SectionTable[ParentSectionNumber].Section
                                 : UndefSection;
    Block *B = nullptr;

```
- **EN**: Implements logic around `getXCOFFSymbolContainingSymbolRef`, `getSymbolIndex`, `getSectionNumber`, `contains`; this block manipulates JITLink graph structures or link-time passes.
- **CN**: 围绕 `getXCOFFSymbolContainingSymbolRef`, `getSymbolIndex`, `getSectionNumber`, `contains` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass。

### Lines 288-298
```cpp
    // TODO: Clean up the logic for handling undefined symbols
    if (!CsectTable.contains(CsectSymbolIndex) && !IsUndefinedSection) {
      object::SectionRef &SectionRef =
          SectionTable[ParentSectionNumber].SectionData;
      auto Data = SectionRef.getContents();
      if (!Data)
        return Data.takeError();
      auto CsectSymbolAddr = CsectSymbol.getAddress();
      if (!CsectSymbolAddr)
        return CsectSymbolAddr.takeError();

```
- **EN**: Implements logic around `getContents`, `takeError`, `getAddress`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `getContents`, `takeError`, `getAddress` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 299-307
```cpp
      ArrayRef<char> SectionBuffer{*Data};
      auto Offset = *CsectSymbolAddr - SectionRef.getAddress();

      LLVM_DEBUG(dbgs() << "      symbol entry: offset = " << Offset
                        << ", size = " << CsectSymbol.getSize()
                        << ", storage class = "
                        << getStorageClassString(CsectSymbol.getStorageClass())
                        << "\n");

```
- **EN**: Implements logic around `getAddress`, `getSize`, `getStorageClassString`.
- **CN**: 围绕 `getAddress`, `getSize`, `getStorageClassString` 实现具体逻辑。

### Lines 308-316
```cpp
      B = &G->createContentBlock(
          *ParentSection, SectionBuffer.slice(Offset, CsectSymbol.getSize()),
          orc::ExecutorAddr(*CsectSymbolAddr), CsectSymbol.getAlignment(), 0);

      CsectTable[CsectSymbolIndex] = B;
    } else {
      B = CsectTable[CsectSymbolIndex];
    }

```
- **EN**: Implements logic around `createContentBlock`, `slice`, `ExecutorAddr`; this block manipulates JITLink graph structures or link-time passes.
- **CN**: 围绕 `createContentBlock`, `slice`, `ExecutorAddr` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass。

### Lines 317-332
```cpp
    Scope S{Scope::Local};
    if (Symbol.getSymbolType() & XCOFF::SYM_V_HIDDEN ||
        Symbol.getSymbolType() & XCOFF::SYM_V_INTERNAL)
      S = Scope::Hidden;
    else if (Global)
      S = Scope::Default;
    // TODO: map all symbols for c++ static initialization to SideEffectOnly

    Linkage L = Weak ? Linkage::Weak : Linkage::Strong;
    auto SymbolAddr = Symbol.getAddress();
    if (!SymbolAddr)
      return SymbolAddr.takeError();
    auto IsCallableOrErr = Symbol.isFunction();
    if (!IsCallableOrErr)
      return IsCallableOrErr.takeError();

```
- **EN**: Implements logic around `getSymbolType`, `getAddress`, `takeError`, `isFunction`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `getSymbolType`, `getAddress`, `takeError`, `isFunction` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 333-343
```cpp
    auto BlockOffset = *SymbolAddr - B->getAddress().getValue();

    LLVM_DEBUG(dbgs() << "      creating with linkage = " << getLinkageName(L)
                      << ", scope = " << getScopeName(S) << ", B = "
                      << format_hex(B->getAddress().getValue(), 16) << "\n");

    SymbolIndexTable[SymbolIndex] =
        &G->addDefinedSymbol(*B, BlockOffset, *SymbolName, Symbol.getSize(), L,
                             S, *IsCallableOrErr, true);
  }

```
- **EN**: Implements logic around `getAddress`, `getScopeName`, `format_hex`, `addDefinedSymbol`; this block manipulates JITLink graph structures or link-time passes.
- **CN**: 围绕 `getAddress`, `getScopeName`, `format_hex`, `addDefinedSymbol` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass。

### Lines 344-354
```cpp
  return Error::success();
}

Error XCOFFLinkGraphBuilder::processRelocations() {
  LLVM_DEBUG(dbgs() << "  Creating relocations...\n");

  for (object::SectionRef Section : Obj.sections()) {
    auto SectionName = Section.getName();
    if (!SectionName)
      return SectionName.takeError();

```
- **EN**: Implements logic around `success`, `processRelocations`, `getName`, `takeError`; this block manipulates JITLink graph structures or link-time passes; handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `success`, `processRelocations`, `getName`, `takeError` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 355-366
```cpp
    LLVM_DEBUG(dbgs() << "    Relocations for section " << *SectionName
                      << ":\n");

    for (object::RelocationRef Relocation : Section.relocations()) {
      SmallString<16> RelocName;
      Relocation.getTypeName(RelocName);
      object::SymbolRef Symbol = *Relocation.getSymbol();

      auto TargetSymbol = Symbol.getName();
      if (!TargetSymbol)
        return TargetSymbol.takeError();

```
- **EN**: Implements logic around `getTypeName`, `getSymbol`, `getName`, `takeError`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `getTypeName`, `getSymbol`, `getName`, `takeError` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 367-384
```cpp
      auto SymbolIndex = Obj.getSymbolIndex(Symbol.getRawDataRefImpl().p);

      LLVM_DEBUG(dbgs() << "      " << format_hex(Relocation.getOffset(), 16)
                        << " (idx: " << SymbolIndex << ")"
                        << " " << RelocName << " " << *TargetSymbol << "\n";);

      assert(SymbolIndexTable.contains(SymbolIndex) &&
             "Relocation needs a record in the symbol table");
      auto *S = SymbolIndexTable[SymbolIndex];
      auto It = find_if(G->blocks(),
                        [Target = orc::ExecutorAddr(Section.getAddress() +
                                                    Relocation.getOffset())](
                            const Block *B) -> bool {
                          return B->getRange().contains(Target);
                        });
      assert(It != G->blocks().end() &&
             "Cannot find the target relocation block");
      Block *B = *It;
```
- **EN**: Implements logic around `getSymbolIndex`, `assert`, `find_if`, `ExecutorAddr`, and 2 more symbols; this block manipulates JITLink graph structures or link-time passes; handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `getSymbolIndex`, `assert`, `find_if`, `ExecutorAddr`, and 2 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 385-400
```cpp

      auto TargetBlockOffset = Section.getAddress() + Relocation.getOffset() -
                               B->getAddress().getValue();
      switch (Relocation.getType()) {
      case XCOFF::R_POS:
        B->addEdge(ppc64::EdgeKind_ppc64::Pointer64, TargetBlockOffset, *S, 0);
        break;
      default:
        SmallString<16> RelocType;
        Relocation.getTypeName(RelocType);
        return make_error<StringError>(
            "Unsupported Relocation Type: " + RelocType, std::error_code());
      }
    }
  }

```
- **EN**: Implements logic around `getAddress`, `addEdge`, `getTypeName`, `make_error<StringError>`, and 1 more symbols; this block manipulates JITLink graph structures or link-time passes; handles relocation, fixup, or symbol-resolution work; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `getAddress`, `addEdge`, `getTypeName`, `make_error<StringError>`, and 1 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，处理重定位、fixup 或符号解析工作，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 401-415
```cpp
  return Error::success();
}

Expected<std::unique_ptr<LinkGraph>> XCOFFLinkGraphBuilder::buildGraph() {
  LLVM_DEBUG(dbgs() << "Building XCOFFLinkGraph...\n");

  // FIXME: Check to make sure the object is relocatable

  if (auto Err = processSections())
    return Err;
  if (auto Err = processCsectsAndSymbols())
    return Err;
  if (auto Err = processRelocations())
    return Err;

```
- **EN**: Implements logic around `success`, `buildGraph`; this block manipulates JITLink graph structures or link-time passes; handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `success`, `buildGraph` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 416-420
```cpp
  return std::move(G);
}

} // namespace jitlink
} // namespace llvm
```
- **EN**: Introduces declarations for `jitlink`, `llvm`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `jitlink`, `llvm` 等声明，定义本文件后续使用的数据结构或接口。

## Key Concepts / 关键概念

- **JITLink graph linking / JITLink 图链接**:
  - **EN**: Represents object code as graphs of blocks and edges so passes can rewrite relocations before final linking
  - **CN**: 把目标代码表示为块与边构成的图，使 pass 能在最终链接前重写重定位
- **Link graphs / 链接图**:
  - **EN**: Represents atoms of code/data and relocation edges explicitly for JIT-time rewriting
  - **CN**: 显式表示代码/数据原子及其重定位边，以便在 JIT 期重写
- **Relocations and fixups / 重定位与 fixup**:
  - **EN**: Bridges symbolic references to concrete addresses or relocation records
  - **CN**: 把符号引用连接到具体地址或重定位记录
- **Debug/unwind metadata / 调试与展开元数据**:
  - **EN**: Emits or manages metadata needed for debugging and stack unwinding
  - **CN**: 输出或管理调试与栈展开所需的元数据

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `XCOFFLinkGraphBuilder.h`, `llvm/ADT/STLExtras.h`, `llvm/BinaryFormat/XCOFF.h`, `llvm/ExecutionEngine/JITLink/JITLink.h`, `llvm/ExecutionEngine/JITLink/ppc64.h`, `llvm/ExecutionEngine/Orc/Shared/ExecutorAddress.h`, `llvm/ExecutionEngine/Orc/Shared/MemoryFlags.h`, `llvm/Object/ObjectFile.h`, `llvm/Object/XCOFFObjectFile.h`, `llvm/Support/Debug.h`, `llvm/Support/Error.h`, `llvm/Support/Format.h` ... (+2 more)
- **LLVM subsystems / LLVM 子系统**: ExecutionEngine, Object, Support, BinaryFormat
